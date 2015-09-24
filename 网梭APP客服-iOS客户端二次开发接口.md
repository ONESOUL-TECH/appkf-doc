# 网梭APP客服 - iOS客户端二次开发接口


## 目录
- [准备工作](#准备工作)
  - [下载客服SDK-iOS](#下载sdk)
  - [SDK支持的iOS版本](#supported-iOS-version)
  - [添加SdK到工程](#添加SDK到工程)
  - [设置编译选项](#设置编译选项)
- [初始化SDK](#初始化SDK)
  - [SDK初始化工作](#SDK初始化工作)
  - [实现QZUCCDelegate接口](#实现QZUCCDelegate接口)
- [IM消息会话](#IM消息会话)
  - [开始IM消息会话](#开始IM消息会话)
  - [APP主动发送IM消息](#APP主动发送IM消息)

## <a id="准备工作"></a>准备工作
#### <a id="下载sdk"></a>1.下载客服SDK-iOS
[下载 Onesoul-IM-SDK for iOS](https://github.com/ONESOUL-TECH/appkf-SDK-iOS)
#### <a id="supported-iOS-version"></a>2.SDK支持的iOS版本
iOS6.0以上版本。兼容iPhone、iPad、iPod touch、模拟器等设备。
#### <a id="添加SDK到工程"></a>3.添加SDK到工程

* 下载完成后，找到SDK目录，里面有:
   * libOnesoul-IM-SDK.a： 静态库文件
   * Onesoul-IM-SDK-Bundle.bundle：静态库依赖的资源文件bundle
   * Onesoul-IM-SDK：头文件

   把SDK下的全部内容加入到你的工程中 

* 添加以下Linked Frameworks and Librarys:
	* libxml2.dylib
	* libz.1.2.5.dylib
	* libc++.dylib
	* libicucore.dylib
	* libresolv.dylib
	* libsqlite3.dylib
	* libOnesoul-IM-SDK.a
* 在你的.m文件中引入sdk头文件：  
	\#import "QZUCC.h"

#### <a id="设置编译选项"></a>4.设置编译选项

* Build Settings -> Other Linker Flags: 增加 -ObjC 选项。
* XCode7编译环境下：Building Settings -> Build Options -> Enable Bitcode: 设置为No
  
## <a id="初始化SDK">初始化SDK

#### <a id="SDK初始化工作"></a>1. SDK初始化工作
在使用客服SDK之前，需要先初始化：

```
- (void)setupUCCSDK {
    // 1
    [QZUCC initWithAbility:@"[\"IM\",\"Phone\"]" bundle:@"Onesoul-IM-SDK-Bundle.bundle" delegate:self];
    
    // 2
    QZUCC *ucc = [QZUCC getInstance];
    [ucc connectWithUCMUrl:@"os.qyucc.com:809" qzId:userId authToken:@"" completion:^{
        
        NSLog(@"MainViewCtrl======授权成功=========");
        
        [self checkAuth];
        
    } fail:^(NSError *error) {
        NSLog(@"MainViewCtrl======授权失败=========%@", error);
    }];    
}

```
解释一下上面的代码：

1. initWithAbility:bundle:delegate

   - 加载SDK，并声明你的APP需要用到的能力类型：
   "IM"表示即时消息能力、"Phone"表示实时语音能力

   - delegate是QZUCCDelegate类型，这里设置为self，所以后续你需要实现QZUCCDelegate；
2. [QZUCC getInstance]得到一个全局唯一的QZUCC实例；
3. 设置完成后，就可以开始连接客服后台服务器:
      connectWithUCMUrl:qzId:authToken:completion:
   这个方法的参数如下：
   * UCMUrl: 客服服务器的地址
   * qzId: 当前用户在客服服务器的用户账号，这个需要事先你的APP后台获取到；
   * authToken: 当前用户在客服服务器的用户token，也是事先从你的APP后台获取到。

上面几行代码，就完成了初始化，然后我们来看QZUCCDelegate的实现:

#### <a id="实现QZUCCDelegate接口"></a>2. 实现QZUCCDelegate接口

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

## <a id="IM消息会话"></a>IM消息会话

#### <a id="开始IM消息会话"></a>1. 开始IM消息会话

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

* 开始IM消息会话之前，需要先从你的后台服务获取对应的IM客服接入号码，如例子代码中的"uckf.200000411201"；
* pushChatFromViewController: 方法，指定将IM消息会话ViewController压入到当前ViewController之上。
* 所有会话过程及IM消息的UI工作都由SDK自动完成，很简单，是不是。

#### <a id="APP主动发送IM消息"></a>2. APP主动发送IM消息
```
[暂不提供]
```
