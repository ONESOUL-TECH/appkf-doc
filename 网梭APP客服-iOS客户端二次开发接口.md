# 网梭APP客服 - iOS客户端二次开发接口


## 目录
- [准备工作](#准备工作)

- [初始化SDK](#初始化SDK)

- [实时语音通话](#实时语音通话)

- [IM消息会话](#IM消息会话)

## 准备工作
#### 1. 下载KFSDK-iOS
[KFSDK-iOS的下载地址](http://xxx)
#### 2. SDK支持的iOS版本
iOS6.0以上版本。兼容iPhone、iPad、iPod touch等设备。
#### 3. 添加Framework

下载完成后，解压得到QZUCC.framework和QZUCC.bundle两个文件，添加这两个文件到你的APP工程，同时引入sdk：

`#import <QZUCC/QZUCC.h>
#### 4. 设置编译选项

为保证Framework能够正常工作，需要在Build Settings -> Other Linker Flags 参数中增加 -ObjC 选项。

  
## 初始化SDK

#### 1. SDK初始化工作
在使用客服SDK之前，需要先初始化：

```
- (void)setupUCCSDK {
    // 1
    [QZUCC initWithAbility:@"[\"IM\",\"Phone\"]" icon:nil delete:self];
    
    // 2
    [[QZUCC getInstance] addCallEventObserver:self];
    [[QZUCC getInstance] setUseCustomCallUI:NO];
    
    // 3
    [[QZUCC getInstance] connectWithUCMUrl:demoUCCServer qzId:demoCustomerUCCID authToken:demoCustomerUCCToken deviceToken:nil completion:^{
        
        NSLog(@"======授权成功=========");
        
    } fail:^(NSError *error) {
        NSLog(@"======授权失败=========%@", error);
    }];
}

```
解释一下上面的代码：

1. initWithAbility: 加载SDK，并声明你的APP需要用到的能力类型："IM"表示即时消息能力、"Phone"表示实时语音能力。delegate是QZUCCDelegate类型，这里设置为self，所以后续你需要实现QZUCCDelegate。
2. [QZUCC getInstance]得到一个全局唯一的QZUCC实例，addCallEventObserver: 设置实时语音呼叫事件的监听者；如果你需要在APP中记录语音呼叫的开始和结束事件，那么就需要实现QCallDelegate.
3. 设置完成后，就可以开始连接客服后台服器:
      connectWithUCMUrl:qzId:authToken:deviceToken:completion:
   这个方法的参数如下：
   * UCMUrl: 客服服务器的地址
   * qzId: 当前用户在客服服务器的用户账号，这个需要从事先你的APP后台获取到；
   * authToken: 当前用户在客服服务器的用户token，也是事先从你的APP后台获取到。
   * deviceToken: APNS消息推送需要用到的token。

上面几行代码，就完成了初始化，然后我们来看QZUCCDelegate的实现:

#### 2. 实现QZUCCDelegate接口

```
 #pragma mark - QZUCCDelegate protocol implementation

/**
 * 获取AppDelegate下root view ctrl
 * @return self.window.rootViewController;
 */
- (UIViewController *)getAppRootViewController {
    return self.window.rootViewController;
}

/**
 * 查询用户
 * @param paramString 参数字符串
 * @param type 参数类型，详见ParamType
 * @return 查询成功返回QUserInfo对象,否则返回nil
 */
- (QUserInfo *)getQUserInfo:(NSString *)paramString type:(ParamType)type {
    QUserInfo *usrInfo = nil;
    if (type == ParamTypeQZId) {
        usrInfo = [[QUserInfo alloc] init];
        //TODO:第三方应用负责补充，QZUCC关联帐户信息查询
        
        usrInfo.qzId = @"1111";
        usrInfo.name = @"客服";
        usrInfo.profileUrl = nil;
    } else if (type == ParamTypePhoneNumber) {
        usrInfo = [[QUserInfo alloc] init];
        //TODO:第三方应用负责补充，如企业通讯录或客户通讯录中查询
        usrInfo.name = @"客服";
    }
    return usrInfo;
}

/**
 * 各能力状态变化事件
 * @param ability 能力类型，参见Ability常量定义
 * @param state 状态值，取值0、1，其中1--初始化完成且连接成功，0则是断开连接
 * @return
 */
- (void)onAbilityStateChange:(NSString *)ability state:(int)state {
}

/**
 * 坐席服务事件
 * @param eventDic  - 事件详情内容
 * @param type - 事件类型
 * @return
 */
- (void)onAgentServiceEventChange:(NSDictionary *)eventDic eventType:(NSString *)type {
}

/**
 * 座席状态变化事件
 * @param agentInfo 座席信息，详见各属性描述
 *
 */
- (void)onAgentStatusEventChange:(QAgentInfo *)agentInfo {
}

/**
 * 指定sender_id订阅来自其所发送的消息，当有指定Id后，sdk接收该sender_id的消息后执行<onRecvMessageFromSubscribeSenderIds:>回调方法
 * 其中sender_id的取值范围：
 * --   1.普通用户，则直接填写qz_id;
 * --   2.后台服务中消息中转代理模块的特定用户，则由后台开发人员指定;
 * 若返回 @[MessageInChatRecvRangeAll], 则订阅所有普通聊天, 但不涉及群聊天;其中MessageInChatRecvRangeAll为常量；
 */
- (NSArray *)subscribeMessageFromSenderIds {
    //1.普通用户，如：200000110004
    //2.特定用户，如：hs
    //return @[@"200000110004",@"hs"];
    
    //3.订阅所有用户
    return @[MessageInChatRecvRangeAll];
}

/*
 * 接收subscribeMessageFromSenderIds中指定sender_id发来的消息
 * @param msgInfo 消息对象
 */
- (void)onRecvMessageFromSubscribeSenderIds:(QMessageRecvInfo *)msgInfo {
}

```

## 实时语音通话
#### 1. 开始一个实时语音通话：

```
- (IBAction)phoneCall:(UIButton *)sender {
    QZUCC *ucc = [QZUCC getInstance];
    
    int qphoneConnect = [ucc getAbilityState:AbilityPhone];
    if (!qphoneConnect) {
        NSLog(@"客服后台连接中,请稍候。。。");
        return;
    }
    
    // 1
    NSString *callCenterPhoneId = @"1111";
    
    // 2
    [ucc dialPhone:callCenterPhoneId completion:^{
        NSLog(@"呼叫CallCenter Phone(%@)", callCenterPhoneId);
    } fail:^(NSError *error) {
        NSLog(@"呼叫CallCenter Phone(%@)失败，err = %@", callCenterPhoneId, error);
    }];
}
```
解释一下：

1. 实时语音通话需要先指定客服接入号码，如例子中的callCenterPhoneId。在实际应用中，APP应用从后台服务获取一个（或多个）语音客服的接入号码；
2. 然后调用dialPhone:completion: 方法。

呼叫开始后，SDK就会显示正在呼叫的UI界面，并且实时更新呼叫接续的过程。开始一个实时语音通话就是这么简单。

#### 2. 实现QCallDelegate接口
如果你希望获取更多的呼叫事件（成功、失败、呼入），就需要实现QCallDelegate接口：

```
 #pragma mark - QCallDelegate implementation

/**
 * 电话呼入通知事件
 * @param callInfo 参见QCallInfo对象
 * @return
 */
- (void)onIncomingCall:(QCallInfo *)callInfo
{
    NSLog(@"<onIncomingCall:>- %@", callInfo.phoneNumber);
}

/**
 * 通话对象状态变化通知事件
 * @param callInfo 参见QCallInfo对象
 * @return
 */
- (void)onCallStateChanged:(QCallInfo *)callInfo
{
    switch (callInfo.callState) {
        case CallStateIncoming://电话呼入事件
            break;
        case CallStateCalling://电话呼出事件，
            break;
        case CallStateConfirmed://呼入或呼出电话接通事件
            break;
        case CallStateDisconnected://电话挂断或被挂断事件
            break;
        default:
            break;
    }
}

```

## IM消息会话

#### 1.开始IM消息会话

```
- (IBAction)chat:(UIButton *)sender {
    QZUCC *ucc = [QZUCC getInstance];
    
    int chtcltConnect = [ucc getAbilityState:AbilityIM];
    if (!chtcltConnect) {
        NSLog(@"客服后台连接中,请稍候。。。");
        return;
    }

    // 1
    QUserInfo *usrInfo = [[QUserInfo alloc] init];
    usrInfo.qzId = @"uckf.200000411201";
    usrInfo.name = @"客服";
    
    // 2
    [ucc pushChatFromViewController:self target:usrInfo];
}
```

解释一下：
1. 开始IM消息会话之前，需要先从你的后台服务获取对应的IM客服接入号码，如例子代码中的"uckf.200000411201"；
2. pushChatFromViewController: 方法，指定将IM消息会话ViewController压入到当前ViewController之上。
3. 所有会话过程都由SDK自动完成，很简单，是不是。

#### 2.APP主动发送IM消息
```
[待完善]
```
