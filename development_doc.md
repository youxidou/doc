# 游戏兜接入开发指南


### SDK下载

- [php](https://github.com/youxidou/game-php-sdk)


### 接入SDK demo

- [php-demo](https://github.com/youxidou/game-php-demo)

### 功能接口
- 登陆
- 支付

### 准备

- 申请app_key, app_secret;
- 充值同步跳转地址
- 充值异步跳转地址

### 接口域名地址


```
http://game.yxd17.com/api
```

### 签名算法

- 签名方式:SHA1加密;
    - 签名字符串为utf-8编码;
    - 不要urlencode的原始数据;
    - 加密值采用signature作为参数字段传递;

```
sha1(签名字符串+app_secret)
```

- 算法:
  - 在请求参数列表中,除去`signature`参数外，其他需要使用到的参数皆是要签名的参数;
  - 所有参数必须按键名升序排序

- 示例:

```php
//签名方法
function signData($data, $secret)
{
    ksort($data);
    $str = urldecode(http_build_query($data)). $secret;

    return sha1($str);
}

$data = array(
    "app_key" => "app_key",
    "timestamp" => 1444801543,
    "nonce" => "JK7M1juIqZkDtI6B",
    "token" => "Cwa55RSzBeV1SHWk",
    "game_url" => "http://gc.hgame.com/home/game/appid/100000/gameid/100116?bar=&from=",
);

$signature = signData($data, "app_secret");
```

- 签名错误信息
  - 保证系统一致性，由于签名验证产生的错误，做以下规范


| Code | 说明  |
| -----| :---------|
| 1001 | 签名验证错误 |
| 1002 | 缺少参数：signature, timestamp, nonce, game_key |
| 1003 | timestamp 超过有效期 |
| 1004 | nonce 重复 |
| 1005 | 没有对应的 secret_key |


### 登陆

- 目的就是为了获取`游戏中心`当前登录的用户信息

##### 传递登陆Token

- 步骤：
  - `游戏中心`通过`iframe`打开游戏链接，并把登陆唯一标识`token`及相关信息做为参数附加在URL中
  - 将得到的token，向平台换取用户信息，完成登录

- 示例：

```
<iframe src="http://www.game-example.com/?app_key=51776cabcdefs&timestamp=1436954315&nonce=7kCbbPMpumx6Qe9N&token=JDCaslecKDSKDSb&game_url=http%3A%2F%2Fgc.hgame.com%2Fhome%2Fgame%2Fappid%2F100000%2Fgameid%2F100126%3Fbar%3D%26from%3D&signature=1007373a88cae35e123123a1f8e70498816669"></iframe>
```

- 参数说明：

| 参数 | 必填  | 描述 |
| -----| :---| :---------|
| app_key | 是 | 这里是游戏中心提供的app key |
| timestamp | 是 | 当前时间戳 |
| nonce | 是 |  随机字符串 |
| token | 是 | 登陆标识 |
| game_url | 是 | 游戏url地址， 获取之后先urldecode再进行签名计算 |
| signature | 是 |  签名 |


##### 验证token，并获取用户信息

- 游戏服务器向`游戏中心`服务端验证token，`游戏中心`返回对应的用户信息

- 请求地址:

```
GET /user/getUserInfo
```

- 参数说明:

| 参数 | 必填  | 描述 |
| -----| :---| :---------|
| app_key | 是 | 这里是游戏中心提供的app key |
| timestamp | 是 | 当前时间戳 |
| nonce | 是 |  随机字符串 |
| token | 是 | 登陆标识 |
| signature | 是 |  签名 |

- 正确返回格式:

```
http code = 200
{
	"open_id":"aaa33dd32dd2dadd",  //用户对游戏唯一id
	"avatar": "http://www.avatar.com/avatar.jpg", //始终是http格式
	"nickname": "魔界大神",
	"union_id":"aaaadddddaaaaaadadad" //用户对开发商唯一id
}
```

- 错误返回格式:

```
http code = 400
{
    "code": 2001, 
    "message":"操作成功",   // 不存在或过期
}
```

- 返回Code说明:

| Code | 说明  |
| -----| :---------|
| 2001 | 不存在或过期 |
| 2002 | 禁止用户登录 |
| 2003 | 游戏已下架 |


### 支付

- 用途:购买游戏内的道具或金币;

##### JS SDK接口发起支付

- 步聚:
  - 引入游戏兜JSSDK `//game.yxd17.com/js/sdk.js`;（注意：不需要添加http，以便自适应http和https协议）
  - 由游戏服务器生成购买订单，并调用YXD.pay方法，发起订单
  - 用户通过游戏中心完成付费
  - `游戏中心`回调游戏提供的notify_url，通知游戏付费成功

- pay接口

```
YXD.pay(pay_data, callback)
```

- pay_data参数说明: 支付信息

| 参数 | 描述 |
| -----|  :---------|
| app_key | 这里是游戏中心提供的app key |
| prepay_id | 统一下单接口返回的prepay_id参数值 |
| timestamp | 当前时间戳 |
| nonce |  随机字符串 |
| signature |  签名 |

- 示例:

```
 <script language="javascript">
    YXD.pay(pay_data, function (result)
    {
        // result结构如下
        result.code
    });
 </script>
```

- result字段说明:

| 参数 | 必填  | 描述 |
| -----| :---| :---------|
| code | 是 | 错误码 |


- 错误码: 

| 值 | 描述 |
| -----| :---------|
| -1 | 用户取消 |
| 0 | 完成支付(通过跳转完成的支付只有异步通知, 没有回调通知) |
| 1 | 参数错误 |
| >0 | 其他错误 |


##### 统一下单接口

- 商户系统需先调用该接口在游戏中心支付服务后台生成预支付交易单，返回正确的预支付交易回话标识后再调用`sdk.pay`接口

- 接口：

```
POST /pay/unified/order
```

- 参数说明

| 参数 | 必填  | 描述 |
| -----| :---| :---------|
| app_key | 是 | 这里是游戏中心提供的app key |
| open_id | 是 | 用户id
| money | 是 | 道具支付金额（单位元），精确到小数点后两位 |
| game_order_no | 是 | 游戏生成的订单号 |
| title | 是 | 游戏道具名称 |
| description | 否 | 游戏道具描述 |
| attach | 否 | （长度：255字符）附加数据，在支付通知中原样返回，可作为自定义参数使用。|
| notify_url | 是 |  支付完成后异步通知URL |
| timestamp | 是 | 当前时间戳 |
| nonce | 是 |  随机字符串 |
| signature | 是 |  签名 |

- 返回结果

| 参数 | 必填  | 描述 |
| -----| :---| :---------|
| result_code | 是 | SUCCESS/FAIL  此字段是通信标识，非交易标识，交易是否成功需要查看result_code来判断  |
| app_key | 是 | 这里是游戏中心提供的app key |
| prepay_id | 是 | 预支付交易会话标识 |
| timestamp | 是 | 当前时间戳 |
| nonce | 是 |  随机字符串 |
| signature | 是 |  签名 |

##### 支付异步回调

- 说明:
  - 采用`POST`方式调用商家的充值异步通知地址(notify_url)
  - 目前只有在支付成功的情况下才会异步回调商家！
  - 商家接到异步通知并且验签无误后，完成商家的业务逻辑(发货等)之后，成功返回纯字符串“SUCCESS”，否则返回其他。
  - 返回其他时，游戏中心会认为商家接收失败，会一直重复发支付成功通知，在交易完成的第（0min, 2min, 10min, 20min, 1hour, 2hour, 6hour, 15hour, 24hour)时段发起请求，直至收到成功标志或者超时。

- 回调返回参数值: `返回所有值都需要参与签名`，可以通过`$_POST`来获取返回值

| 参数 | 描述 |
| -----|  :---------|
| result_code | SUCCESS/FAIL  此字段是通信标识，非交易标识，交易是否成功需要查看result_code来判断  |
| app_key | 这里是游戏中心提供的app key |
| open_id | 用户id |
| notify_time | 通知时间 |
| notify_id | 通知id |
| money | 道具支付金额（单位元），精确到小数点后两位 |
| title | 订单标题 |
| description | 订单描述 |
| attach |（不是必须，长度：255字符） 商家数据包，原样返回，如果创建预付订单时填写该值，则通知时，该值原样返回 |
| trade_no | 游戏兜的交易订单号 |
| game_order_no | 游戏生成的订单号 |
| timestamp | 当前时间戳 |
| nonce |  随机字符串 |
| signature |  签名 |
