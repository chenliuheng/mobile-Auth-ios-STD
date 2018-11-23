# 1. 开发环境配置 

## 1.1. 环境配置及发布

1. 导入统一认证framework

直接将统一认证`TYRZNoUISDK.framework`拖到项目中

![iOS-1](image/iOS-1.png)

2. 在Xcode中找到`TARGETS-->Build Setting-->Linking-->Other Linker Flags`在这选项中需要添加`-ObjC`

![iOS-4](image/iOS-4.jpg)

## 1.2. Hello 统一认证 

本节内容主要面向新接入统一认证的开发者，介绍快速集成统一认证的基本服务的方法。

### 1.2.1. 统一认证登录流程

![](image/mobile_auth.png)

由流程图可知，业务客户端集成SDK后只需要完成2步集成实现登录

1. 调用SDK方法来获得token；
2. 携带token通过业务服务端到认证服务端的本机号码校验接口，进行号码校验

### 1.2.2. 统一认证登录集成步骤

**第一步：**

初始化代码只需要执行一次就可以。

```objective-c
self.login = [TYRZLogin loginWithAppId:APPID appKey:APPKEY];
```

**第二步：**

在需要用到登录的地方调用登录接口即可，以下是登录示例

```objective-c
- (void)showImplicitLogin {
    [self.login requestTokenWithTimeout:8000 Complete:^(NSDictionary * _Nonnull    loginResponse) {

        weakself.maskView.hidden = YES;
        [weakself.indicatorView stopAnimating];

        NSString *resultCode = loginResponse[@"resultCode"];
        weakself.token = loginResponse[@"token"];
        NSMutableDictionary *result = [NSMutableDictionary dictionaryWithDictionary:loginResponse];
        if ([resultCode isEqualToString:CLIENTSUCCESSCODECLIENT]) {
            result[@"result"] = @"获取token成功";
        } else {
            result[@"result"] = @"获取token失败";
        }
        [weakself showInfo:result];
    }];
}
```

<div STYLE="page-break-after: always;"></div>

#2. SDK方法描述
## 2.1. 获取token

### 2.1.1 方法描述

开发者向统一认证服务器获取临时凭证`token`。</br>

**token：**开发者服务端可凭临时凭证token通过3.1本机号码校验接口对本机号码进行验证。

</br>

**原型**

`TYRZLogin -> requestTokenWithTimeout`

```objective-c
- (void)requestTokenWithTimeout:(NSTimeInterval)duration Complete:(void(^)(NSDictionary *))complete;
```



### 2.1.2 参数说明

**请求参数**

| 参数       | 类型            | 说明              | 是否必填 |
| -------- | ------------- | --------------- | ---- |
| duration    | NSTimeInterval     | 请求时间  | 是    |
| complete | UAFinishBlock | 登录回调            | 是    |


**响应参数**


| 参数          | 类型         | 说明                                       | 是否必填  |
| ----------- | ---------- | ---------------------------------------- | ----- |
| resultCode  | NSUinteger | 返回相应的结果码。具体响应码见4.1. 本机号码校验接口返回码          | 是     |
| token       | NSString   | 成功时返回：临时凭证，token有效期2min，一次有效，同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 | 成功时必填 |
| resultDesc        | NSString   | 调用描述                                     | 否     |



### 2.1.3 示例

**请求示例代码**


```objective-c
/**
 获取token
 */
 - (IBAction)loginimplicit:(id)sender {
 
    self.maskView.hidden = NO;
    [self.indicatorView startAnimating];
    typeof(self) weakself = self;
 
    [self.login requestTokenWithTimeout:8000 Complete:^(NSDictionary * _Nonnull loginResponse) {
 
        weakself.maskView.hidden = YES;
        [weakself.indicatorView stopAnimating];
 
        NSString *resultCode = loginResponse[@"resultCode"];
        weakself.token = loginResponse[@"token"];
        NSMutableDictionary *result = [NSMutableDictionary dictionaryWithDictionary:loginResponse];
        if ([resultCode isEqualToString:CLIENTSUCCESSCODECLIENT]) {
            result[@"result"] = @"获取token成功";
        } else {
            result[@"result"] = @"获取token失败";
        }
        [weakself showInfo:result];
 }];
 
 }
```


**响应示例代码**

```
{
    resultDesc = "";
    resultCode = 103000;
    token = STsid00000015087457254472qa7Mh1AAZH1U0xwvoMnKu5XxipjWXWE;
}
```

<div STYLE="page-break-after: always;"></div>

# 3. 平台接口说明

## 3.1. 本机号码校验接口

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起本机号码校验接口的调用。

调用本接口，必须保证：

1. token在有效期内（2分钟）。
2. token还未使用过。
3. 应用服务器出口IP地址在开发者社区中配置正确。

对于本机号码校验，需要注意：

1. 本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。
2. 签订合同后，将不在提供每天免费的测试次数。

### 3.1.1. 业务流程

SDK在获取token过程中，用户手机必须在打开数据网络情况下才能获取成功。**注：本业务目前仅支持中国移动号码，建议开发者在使用该功能前，判断当前用户手机运营商**

![](image/mobile_auth.png)

</br>

### 3.1.2. 接口说明

**调用次数说明：**本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。

**请求地址：** https://www.cmpassport.com/openapi/rs/tokenValidate

**协议：** HTTPS

**请求方法：** POST+json

**回调地址：**请参考开发者接入流程文档

</br>

### 3.1.3.  参数说明

*1、json形式的报文交互必须是标准的json格式；*

*2、发送时请设置content type为 application/json*

**请求参数**

| 参数          | 类型   | 层级  | 约束                         | 说明                                                         |
| ------------- | ------ | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    |        | **1** | 必选                         |                                                              |
| version       | string | 2     | 必选                         | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | string | 2     | 必选                         | 使用UUID标识请求的唯一性                                     |
| timestamp     | string | 2     | 必选                         | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | string | 2     | 必选                         | 应用ID                                                       |
| **body**      |        | **1** | 必选                         |                                                              |
| openType      | String | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | String | 2     | 是                           | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | String | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | String | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| phoneNum      | String | 2     | 是                           | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | String | 2     | 是                           | 身份标识，字符串形式的token                                  |
| sign          | String | 2     | 是                           | 签名，HMACSHA256( appId + msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |
|               |        |       |                              |                                                              |

**响应参数**

| 参数         | 层级  | 类型   | 约束 | 说明                                                         |
| ------------ | ----- | :----- | :--- | :----------------------------------------------------------- |
| **header**   | **1** |        | 必选 |                                                              |
| msgId        | 2     | string | 必选 | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | string | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | string | 必选 | 应用ID                                                       |
| resultCode   | 2     | string | 必选 | 规则参见4.1平台返回码                                        |
| **body**     | **1** |        | 必选 |                                                              |
| resultDesc   | 2     | String | 必选 | 描述参见4.1平台返回码                                        |
| message      | 2     | String | 否   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams | 2     | String | 否   | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
|              |       |        |      |                                                              |

</br>

### 3.1.4. 示例

**请求示例**

```
{
    "header":{
        "appId":"3000*****401",
        "timestamp":"20180104090953788",
        "version":"1.0",
        "msgId":"8ADFF305-C7FC-B3E1-B1AE-CC130792FBD0"
    },
    "body":{
        "openType":"1",
        "token":"STsid0000001515028196605yc1oYNTuPlTlLT10AR3ywr2WApEq14JH",
        "sign":"227716D80112F953632E4AFBB71C987E9ABF4831ACDA5A7464E2D8F61F0A9477",
     "phoneNum":"38D19FF8CE10416A6F3048467CB6F7D57A44407CB198C6E8793FFB87FEDFA9B8",
        "requesterType":"0"
    }
}
```



**响应示例**

```
{
    "body":{
        "message":"",
        "resultDesc":"是本机号码"
    },
    "header":{
        "appId":"3000*****40",
        "msgId":"8ADFF305-C7FC-B3E1-B1AE-CC130792FBD0",
        "resultCode":"000",
        "timestamp":"20180104090957277"
    }
}
```

<div STYLE="page-break-after: always;"></div>


# 4. 平台返回码说明

## 4.1. 本机号码校验接口返回码

本返回码表仅针对`本机号码校验接口`使用

| 返回码    | 说明              |
| ------ | --------------- |
| 000    | 是本机号码（纳入计费次数）   |
| 001    | 非本机号码（纳入计费次数）   |
| 002    | 取号失败            |
| 003    | 调用内部token校验接口失败 |
| 004    | 加密手机号码错误        |
| 102    | 参数无效            |
| 124    | 白名单校验失败         |
| 302    | sign校验失败        |
| 303    | 参数解析错误          |
| 606    | 验证Token失败       |
| 999    | 系统异常            |
| 102315 | 次数已用完           |




## 4.2. SDK返回码说明

| 错误编号   | 返回码描述     |
| ------ | --------- |
| 103000 | 成功        |
| 200009 | 应用合法性校验失败 |
| 200014 | 手机号码格式错误 |
| 200021 | 数据解析异常 |
| 200022 | 无网络 |
| 200023 | 请求超时 |
| 200027 | 蜂窝网络未开启或者蜂窝网络不稳定 |
| 200028 | 请求出错 |
| 200029 | 请求出错,上次请求未完成 |
| 200030 | 没有初始化参数|
|200031 | 生成token失败 |
| 200033|  复用中间件获取Token失败 |
| 200036| 预取号失败 |
| 200034| 预取号token失效 |
| 200036| 预取号失败 |
| 200048 | 无SIM卡|
| 200050 | EOF异常，网络请求无数据流返回 |
| 200062 | 不支持联通取号 |
| 200063 | 不支持电信取号 |
| 200051 | socket请求检测不到手机数据网络 |
| 200052 | setsockopt创建失败 |
| 200053 | socket连接hostAddress出错 |
| 200054 | socket在CFRunLoop里请求超时 |
| 200055 | socket在CFRunLoop里连接服务器失败 |
| 200056 | socket在CFSocketSendData时请求超时 |
| 200057 | socket在CFSocketSendData时请求出错 |
| 200058 | socket 拼接响应数据时出错 |
| 200059 | Socket无响应 |



<div STYLE="page-break-after: always;"></div>


