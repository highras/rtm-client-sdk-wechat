# fpnn rtm sdk wechat #

#### 使用示例

```javascript
var rtm = require('path/to/rtm-wechat-sdk/rtm.js');
var fpnn = require('path/to/rtm-wechat-sdk/fpnn.js');

let client = new rtm.RTMClient({ 
    ssl_endpoint: 'RTM_ENDPOINT', // 从云上曲率官网控制台获取
    autoReconnect: true,
    connectionTimeout: 10 * 1000,
    pid: 80000071 // 从云上曲率官网控制台获取
});

// ErrorRecorder仅用于异常信息的收集调试，请勿在该事件回调中进行重连等相关操作
client.on('ErrorRecorder', function(err) {
    console.error("on ErrorRecorder");
    console.error(err);
});

client.on('ReloginCompleted', function(successful ,retryAgain, errorCode, retriedCount) {
    console.log("ReloginCompleted, successful: " + successful + " retryAgain: " + retryAgain + " errorCode: " + errorCode + " retriedCount: " + retriedCount);
});

client.on('SessionClosed', function(errorCode) {
    console.log("SessionClosed, errorCode: " + errorCode);
    if (errorCode == rtm.RTMConfig.ERROR_CODE.RTM_EC_INVALID_AUTH_TOEKN) {
        // token error, need to get a new token and login again.
    }
});

client.login(new rtm.RTMConfig.Int64(uid), token, function(ok, errorCode) {

    if (errorCode == fpnn.FPConfig.ERROR_CODE.FPNN_EC_OK) {

        if (!ok) {
            // token error, need to get a new token
            return;
        } else {
            // login successfully
        }

    } else {
        // login error
    }
}, 60 * 1000);

//push service
let pushName = rtm.RTMConfig.SERVER_PUSH.recvMessage;
client.processor.on(pushName, function(data) {

    console.log('\n[PUSH] ' + pushName + ':\n', data);
    // console.log(data.mid.toString());
});

//send message 
client.sendMessage(new rtm.RTMConfig.Int64(123789), 8, 'hello !', '', new rtm.RTMConfig.Int64(0), 10 * 1000, function(err, data) {

    if (err) {

        if (err.hasOwnProperty('mid')) {

            console.error('\n mid:' + err.mid.toString(), err.error);
            return;
        }

        console.error('\n ', err);
    }

    if (data) {

        if (data.hasOwnProperty('mid')) {

            console.log('\n mid:' + data.mid.toString(), data.payload);
            return;
        }

        console.log('\n ', data);
    }
});

// destroy
// client.destroy();
```

#### Events ####

* `event`:
    * `ErrorRecorder`: 内部异常信息输出
        * `err`: **(FPError)** 异常错误结构
    
ErrorRecorder事件只用于异常信息的记录，可将SDK内部产生的一些异常状态和流程输出用于问题排查，请勿在该事件中进行任何其他操作，如重连等。

* `ReloginCompleted`: 重连完成事件
    * `successful`: **(Bool)** 重连是否成功
    * `retryAgain`: **(Bool)** 是否会进行下一次重连尝试
    * `errorCode`: **(Int)** 本次重连收到的错误码
    * `retriedCount`: **(Int)** 已经尝试重连了多少次

在进行自动重连时，每次重连完成都会触发ReloginCompleted事件。

* `SessionClosed`: 连接关闭事件
    * `errorCode`: **(Int)** 造成连接关闭的错误码

#### API ####
* `constructor(options)`: 构造RTMClient
    * `options.endpoint`: **(Optional | string)** rtmGated服务地址, RTM提供
    * `options.ssl_endpoint`: **(Optional | string)** ssl加密rtmGated服务地址, RTM提供
    * `options.pid`: **(Required | number)** 应用编号, RTM提供
    * `options.autoReconnect`: **(Optional | bool)** 是否自动重连, 默认: `true`
    * `options.connectionTimeout`: **(Optional | number)** 超时时间(ms), 默认: `30 * 1000`
    * `options.maxPingIntervalSeconds`: **(Optional | number)** 心跳检查时间，超过多少秒没收到心跳既认为连接断开, 默认: `60`
    * `options.attrs`: **(Optional | object[string, string])** 设置用户端信息, 保存在当前链接中, 客户端可以获取到
    * `options.platformImpl`: **(Optional | Object)** 平台相关接口注入, 默认: `new fpnn.BrowserImpl()`, 微信小程序： `new fpnn.WechatImpl()`
    * `options.md5`: **(Optional | function)** `md5`字符串加密方法
    * `options.regressiveStrategy`: **(Optional | Object)** 退行性重连策略

参数endpoint和ssl_endpoint必须至少存在一个, ssl_endpoint代表加密连接方式，当存在ssl_endpoint时忽略endpoint

* `options.regressiveStrategy`: 退行性重连策略
    * `startConnectFailedCount`: 连接失败多少次后，开始退行性处理,默认3
    * `maxIntervalSeconds`: 退行性重连最大时间间隔,默认8
    * `linearRegressiveCount`: 从第一次退行性连接开始，到最大链接时间，允许尝试几次连接，每次时间间隔都会增大,默认4

* `processor`: **(RTMProcessor)** 监听PushService的句柄

* `login(uid, token, callback, timeout)`: 连接并登陆 

* `destroy()`: 断开连接并开始销毁

* `close()`: 断开连接

#### RTM System Functions

Please refer [RTM System Functions](docs/System.md)

#### Chat Functions

Please refer [Chat Functions](docs/Chat.md)

#### Value-Added Functions

Please refer [Value-Added Functions](docs/ValueAdded.md)

#### Messages Functions

Please refer [Messages Functions](docs/Messages.md)

#### Files Functions

Please refer [Files Functions](docs/Files.md)

#### Friends Functions

Please refer [Friends Functions](docs/Friends.md)

#### Groups Functions

Please refer [Groups Functions](docs/Groups.md)

#### Rooms Functions

Please refer [Rooms Functions](docs/Rooms.md)

#### Users Functions

Please refer [Users Functions](docs/Users.md)

#### Data Functions

Please refer [Data Functions](docs/Data.md)