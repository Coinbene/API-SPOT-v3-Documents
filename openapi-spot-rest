# coinbene-exchange-rest 接口说明




## 基本信息

- REST接口baseurl: https://openapi-exchange.coinbene.com
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题。
- 具体的错误码及其解释在错误代码汇总。

- GET方法的接口, 参数必须在query string中发送。
- POST方法的接口, 参数在 request body中发送(content type application/json)。
- 对参数的顺序不做要求。

## 访问限制

- 当访问接口超过频率限制时，将返回429状态, 请求太频繁。
- 当返回405状态, 请求太频繁IP被封.
- 限制规则，若传入有效的API key则用user id限速；没有的话则拿用户的公网IP限速。

## 接口类型

- 主要为两类接口，公共接口和私有接口。
- 公共接口无需认证即可调用。
- 每个私有请求必须使用规范的验证形式进行签名。私有接口需要使用您的API key进行验证。

## 签名方式

所有接口请求头必须包含以下内容：

- ACCESS-KEY 字符串类型的API key
- ACCESS-SIGN 使用Hex生成字符串
- ACCESS-TIMESTAMP 发起请求的时间戳
- 所有请求都应该含有application/json类型内容，并且是有效的JSON。

ACCESS-SIGN的值生成规则：

- 按照timestamp + method + requestPath + body字符串（+表示字符串连接），以及secret，使用HMAC SHA256方法加密，最后把加密串的字节数组转成字符串返回。
- 其中，timestamp的值与ACCESS-TIMESTAMP请求头相同，必须是UTC时区Unix时间戳的十进制秒数或ISO8601标准的时间格式，精确到毫秒。
- Method是请求方法，字母全部大写：GET/POST
- requestPath是请求接口路径，例如：/api/v3/instrument/depth
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"instrument_id":"BTC","order_id":"xxxx"}
- secret为用户申请API时所生成的
- 任何时候都请不要把secret透露给其他人或传输到服务器端

接口请求样例：

- GET协议接口两种情况: 

```
1. 不带参数：
preHash String：2021-01-07T07:30:07.709ZGET/api/v3/spot/instruments/trade_pair_list
2. 带参数：
preHash String：2021-01-05T03:19:44.727ZGET/api/v3/spot/instruments/trade_pair_one?instrument_id=BTC
```




- POST协议接口情况：

```
preHash String：2021-01-07T09:22:36.443ZPOST/api/v3/spot/order{"instrument_id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
```

- 签名算法验证：


```
源串：2021-01-07T09:22:36.443ZPOST/api/v3/spot/order{"instrument_id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
secret：5a3b727e4cd241af80561d0d415b6edc
生成sign串：38b6ca28291cc6ff1a57de6141783d9d4ed3a182850c71afd3f18ed889541ebf

样例代码（Java版本）：
**
   * 生成签名
   *
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
  private String signForContractOpenApi(String method, String requestUrl, String requestBody, String secret) {
    final DateTimeFormatter utcFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
    String timestamp = utcFormatter.format(ZonedDateTime.now(ZoneOffset.UTC));
    //NEVER use ZonedDateTime.now(ZoneOffset.UTC).toString(), 
    //which may produce timestamp like 'yyyy-MM-dd'T'HH:mm:ss.SSSSSS'Z', but it depends.
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    return signStr;
  }

  /**
   * sha256_HMAC加密
   *
   * @param resource 签名源字符串
   * @param secret   秘钥
   * @return 加密后字符串
   */
  private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    return hash;
  }

  /**
   * 将加密后的字节数组转换成字符串
   *
   * @param bytes 字节数组
   * @return 字符串
   */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
        // 一位补零
        buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }

样例代码（Python版本）：

import hashlib
import hmac
import unittest

def sign(message, secret):
    """
    gen sign
    :param message: message wait sign
    :param secret:  secret key
    :return:
    """
    secret = secret.encode('utf-8')
    message = message.encode('utf-8')
    sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    return sign

class TestUtil(unittest.TestCase):
    def test_sign(self):
        sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")



样例代码（Kotlin版本）：
private fun ByteArray.toHex() = this.joinToString(separator = "") { it.toInt().and(0xff).toString(16).padStart(2, '0') }
private fun String.sha256(secretKey: String): ByteArray = HmacUtils.hmacSha256(secretKey, this)
private val utcFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
**
   * 生成签名
   *
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
fun signForContractOpenApi(method: String, requestUrl: String, requestBody: String, secret:String) {
    //NEVER use ZonedDateTime.now(ZoneOffset.UTC).toString(), 
    //which may produce timestamp like 'yyyy-MM-dd'T'HH:mm:ss.SSSSSS'Z', but it depends.
    val timestamp = utcFormatter.format(ZonedDateTime.now(ZoneOffset.UTC))
    retrun "$timestamp${method.toUpperCase()}$requestUrl${body ?: ""}"
                .sha256(apiSecret)
                .toHex()
}
```


## 接口规范

### 公共接口-获取币对列表

```
获取交易所全部币对信息列表
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/trade_pair_list
```

请求参数：

无

返回字段说明：

| 名称              | 类型   | 说明                   |      |
| ----------------- | ------ | ---------------------- | ---- |
| trade_pair_name   | string | 币对名称， 如 1BTC/BTC |      |
| base_asset        | string | 基础资产，如 1BTC      |      |
| quote_asset       | string | 计价资产，如 BTC       |      |
| price_precision   | string | 价格精度               |      |
| amount_precision  | string | 数量精度               |      |
| taker_fee_rate    | string | taker手续费率          |      |
| maker_fee_rate    | string | maker手续费率          |      |
| min_amount        | string | 委托数量最小限制       |      |
| price_fluctuation | string | 价格波动限制           |      |



```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/trade_pair_list
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T07:30:07.709ZGET/api/v3/spot/instruments/trade_pair_list

Response:
{
    "code":200,
    "data":[
        {
            "trade_pair_name":"1BTC/BTC",
            "base_asset":"1BTC",
            "quote_asset":"BTC",
            "price_precision":"2",
            "amount_precision":"2",
            "taker_fee_rate":"0.1",
            "maker_fee_rate":"0.1",
            "min_amount":"0.1",
            "price_fluctuation":"0.50"
        },
        {
            "trade_pair_name":"ABBC/BTC",
            "base_asset":"ABBC",
            "quote_asset":"BTC",
            "price_precision":"8",
            "amount_precision":"2",
            "taker_fee_rate":"0.001",
            "maker_fee_rate":"0.001",
            "min_amount":"1",
            "price_fluctuation":"0.50"
        },
        ...
    ]
}
```

### 公共接口-单个币对信息

```
获取交易所单个币对信息
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/trade_pair_one?instrument_id=BTC%2FUSDT
```

请求参数：

| 名称          | 类型   | 说明                   |      |
| ------------- | ------ | ---------------------- | ---- |
| instrument_id | string | 币对名称， 如 BTC/USDT |      |



返回字段说明：

| 名称              | 类型   | 说明                   |
| ----------------- | ------ | ---------------------- |
| trade_pair_name   | string | 币对名称， 如 BTC/USDT |
| base_asset        | string | 基础资产，如 BTC       |
| quote_asset       | string | 计价资产，如 USDT      |
| price_precision   | string | 价格精度               |
| amount_precision  | string | 数量精度               |
| taker_fee_rate    | string | taker手续费率          |
| maker_fee_rate    | string | maker手续费率          |
| min_amount        | string | 委托数量最小限制       |
| price_fluctuation | string | 价格波动限制           |




```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/trade_pair_one?instrument_id=BTC%2FUSDT
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T07:36:29.790ZGET/api/v3/spot/instruments/trade_pair_one?instrument_id=BTC%2FUSDT

Response:
{
    "code":200,
    "data":{
        "trade_pair_name":"BTC/USDT",
        "base_asset":"BTC",
        "quote_asset":"USDT",
        "price_precision":"2",
        "amount_precision":"4",
        "taker_fee_rate":"0.0015",
        "maker_fee_rate":"0.013",
        "min_amount":"0.004",
        "price_fluctuation":"0.20"
    }
}
```

### 公共接口-获取深度

```
获取交易所现货深度列表
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/depth?instrument_id=BTC%2FUSDT&depth=5
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                         |
| ------------- | ------ | -------- | ---------------------------- |
| instrument_id | string | 是       | 币对名称，如BTC/USDT         |
| depth         | string | 是       | 深度档位，值有5、10、50、100 |

返回字段说明：

| 名称      | 类型   | 说明                       |
| --------- | ------ | -------------------------- |
| asks      | array  | 卖方深度，[档位价格，数量] |
| bids      | array  | 买方深度，[档位价格，数量] |
| timestamp | string |                            |


```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/depth?instrument_id=BTC%2FUSDT&depth=5
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T07:49:31.732ZGET/api/v3/spot/instruments/depth?instrument_id=BTC%2FUSDT&depth=5

Response:
{
    "code":200,
    "data":{
        "asks":[
            [
                "36779.79",
                "0.1237"
            ],
            [
                "36787.16",
                "0.0474"
            ],
            [
                "36788.17",
                "0.4454"
            ],
            [
                "36794.49",
                "0.0229"
            ],
            [
                "36801.84",
                "0.9221"
            ]
        ],
        "bids":[
            [
                "36728.33",
                "0.0217"
            ],
            [
                "36720.99",
                "0.0221"
            ],
            [
                "36713.63",
                "0.1577"
            ],
            [
                "36706.28",
                "0.0876"
            ],
            [
                "36698.93",
                "0.0163"
            ]
        ],
        "timestamp":"2021-01-07T07:49:32.836Z"
    }
}
```

### 公共接口-获取ticker列表信息

```
获取交易所现货全部ticker的最新成交价、买一价、卖一价和24交易量
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/ticker_list
```

请求参数：无


返回字段说明：

| 名称              | 类型   | 说明                 |
| ----------------- | ------ | -------------------- |
| trade_pair_name   | string | 币对名称，如BTC/USDT |
| last_price        | string | 最新价               |
| lowest_ask        | string | 卖一价               |
| highest_bid       | string | 买一价               |
| highest_price_24h | string | 24h最高价            |
| lowest_price_24h  | string | 24h最低价            |
| volume24h         | string | 24h成交量            |
| chg24h            | string | 24h涨跌幅            |
| chg0h             | string | 0h涨跌幅             |
| amount24h         | string | 24h交易额            |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/ticker_list
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T07:50:34.485ZGET/api/v3/spot/instruments/ticker_list

Response:
{
    "code":200,
    "data":[
        {
            "trade_pair_name":"CPU/USDT",
            "last_price":"0.007600",
            "highest_bid":"0.000000",
            "lowest_ask":"0.000000",
            "highest_price_24h":"0.007600",
            "lowest_price_24h":"0.007600",
            "volume24h":"0.000000",
            "chg24h":"0.00%",
            "chg0h":"0.00%",
            "amount24h":"0.000000"
        },
        ...
    ]
}
```

 

### 公共接口-获取指定的ticker信息

```
获取交易所现货指定ticker的最新成交价、买一价、卖一价和24交易量
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/ticker_one?instrument_id=BTC%2FUSDT
```

请求参数：无

| 名称          | 类型   | 是否必填 | 说明                 |
| ------------- | ------ | -------- | -------------------- |
| instrument_id | string | 是       | 币对名称，如BTC/USDT |

返回字段说明：

| 名称              | 类型   | 说明      |
| ----------------- | ------ | --------- |
| trade_pair_name   | string | 币对名称  |
| last_price        | string | 最新价    |
| lowest_ask        | string | 卖一价    |
| highest_bid       | string | 买一价    |
| highest_price_24h | string | 24h最高价 |
| lowest_price_24h  | string | 24h最低价 |
| volume24h         | string | 24h成交量 |
| chg24h            | string | 24h涨跌幅 |
| chg0h             | string | 0h涨跌幅  |
| amount24h         | string | 24h交易额 |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/ticker_one?instrument_id=BTC%2FUSDT
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T07:53:36.493ZGET/api/v3/spot/instruments/ticker_one?instrument_id=BTC%2FUSDT


Response:
{
    "code":200,
    "data":{
        "trade_pair_name":"BTC/USDT",
        "last_price":"36849.13",
        "highest_bid":"36827.87",
        "lowest_ask":"36879.47",
        "highest_price_24h":"37686.16",
        "lowest_price_24h":"33630.27",
        "volume24h":"4363928722.84",
        "chg24h":"6.23%",
        "chg0h":"6.58%",
        "amount24h":"4363928722.84"
    }
}
```

### 公共接口-获取K线数据

```
获取现货K线数据。K线数据最多可获取2000条。
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/candles?instrument_id=BTC%2FUSDT&period=1&start_time=&end_time=
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                             |      |
| ------------- | ------ | -------- | -------------------------------- | ---- |
| instrument_id | string | 是       | 币对名称，如BTCUSDT              |      |
| start_time    | string | 是       | 开始时间，ISO8601格式时间戳 到秒 |      |
| end_time      | string | 是       | 截止时间，ISO8601格式时间戳 到秒 |      |
| period        | string | 是       | Kline粒度，取值范围参考说明      |      |


```
resolution的值只能取["1", "3", "5", "15", "30",
"60", "120", "240", "360", "720", "D", "W", "M"]，否则请求将会被拒绝，
分别对应 [1min, 3min, 5min, 15min, 30min, 
1hour, 2hour, 4hour, 6hour, 12hour, 1day, 1week, 1month]的时间段
```


返回：


```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/candles?instrument_id=BTC%2FUSDT&period=1&start_time=&end_time=
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:08:52.984ZGET/api/v3/spot/instruments/candles?instrument_id=BTC%2FUSDT&period=1&start_time=&end_time=
		
		
Response:
格式说明:[timestamp,open,high,low,close,volume]
{
    "code":200,
    "data":[
        [
            "2021-01-07T08:08:00.000Z",
            "36950.56",
            "36996.13",
            "36946.35",
            "36980.25",
            "42.5518"
        ],
        [
            "2021-01-07T08:07:00.000Z",
            "37007.96",
            "37007.96",
            "36949.97",
            "36950.56",
            "87.8897"
        ],
        ...
    ]
}
```

### 公共接口-查询最新成交信息

```
获取现货的最新成交信息
限速规则：5次/1秒
HTTP GET /api/v3/spot/instruments/trade_list?instrument_id=BTC%2FUSDT
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                |
| ------------- | ------ | -------- | ------------------- |
| instrument_id | string | 是       | 币对名称，如BTCUSDT |

返回字段说明：


```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/trade_list?instrument_id=BTC%2FUSDT
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:13:34.233ZGET/api/v3/spot/instruments/trade_list?instrument_id=BTC%2FUSDT

格式说明:[trade_pair_name,price,volume,side,timestamp]
Response:
{
    "code":200,
    "data":[
        [
            "BTC/USDT",
            "36906.04",
            "1.1161",
            "sell",
            "2021-01-07T08:13:34.000Z"
        ],
        [
            "BTC/USDT",
            "36907.39",
            "1.3790",
            "buy",
            "2021-01-07T08:13:29.000Z"
        ],
        ...
    ]
}
```

### 公共接口-获取法币和USD汇率

```
获取平台提供的汇率接口
限速规则：1次/1秒
HTTP GET /api/v3/spot/instruments/rate_list
```

请求参数：
无


返回字段说明：

| 名称      | 类型   | 说明              |
| --------- | ------ | ----------------- |
| symbol    | string | 资产币对          |
| rate      | string | 对USD的利率       |
| timestamp | string | 请求时间,国际时间 |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/instruments/rate_list
Method: GET
Headers: 
Accept: application/json
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:16:21.638ZGET/api/v3/spot/instruments/rate_list

Response:
{
    "code":200,
    "data":[
        {
            "symbol":"USD_USDT",
            "rate":"0.9996",
            "timestamp":"2021-01-05T03:57:08.859Z"
        },
        {
            "symbol":"USD_CNY",
            "rate":"6.4570",
            "timestamp":"2021-01-05T03:57:08.859Z"
        },
        {
            "symbol":"USD_KRW",
            "rate":"1085.4800",
            "timestamp":"2021-01-05T03:57:08.859Z"
        },
        {
            "symbol":"USD_JPY",
            "rate":"103.1400",
            "timestamp":"2021-01-05T03:57:08.859Z"
        },
        {
            "symbol":"USD_BRL",
            "rate":"5.2968",
            "timestamp":"2021-01-05T03:57:08.859Z"
        },
        {
            "symbol":"USD_ARP",
            "rate":"84.5210",
            "timestamp":"2021-01-05T03:57:08.859Z"
        }
    ]
}

```



### 私有接口-查询全部账户信息

```
获取用户现货资产的全部账户信息
限速次数：3次/1秒
HTTP GET /api/v3/spot/account/list
```

请求参数 无

返回结果参数

| 名称           | 类型   | 说明     |
| -------------- | ------ | -------- |
| asset          | string | 资产名称 |
| available      | string | 可用余额 |
| frozen_balance | string | 冻结余额 |
| total_balance  | string | 总额     |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/account/list
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: d7409a21176bac4a8f288165c11373e0180d9059b6a9c8ab57a451add0f78360
ACCESS-TIMESTAMP: 2021-01-07T08:42:27.987Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:42:27.987ZGET/api/v3/spot/account/list


Response:
{
    "code":200,
    "data":[
        {
            "asset":"MOAC",
            "available":"9990.00000000",
            "frozen_balance":"8.30000000",
            "total_balance":"9998.30000000"
        },
        {
            "asset":"LTC",
            "available":"9964.99999733",
            "frozen_balance":"0",
            "total_balance":"9964.99999733"
        },
        ...
    ]
}
```

### 私有接口-查询指定账户资产信息

```
获取现货用户指定资产的账户信息
限速次数：6次/1秒
HTTP GET /api/v3/spot/account/one?asset=USDT
```

请求参数

| 名称  | 类型   | 是否必填 | 说明                  |
| ----- | ------ | -------- | --------------------- |
| asset | string | 是       | 资产名称/缩写，如USDT |

返回结果参数

| 名称           | 类型   | 说明            |
| -------------- | ------ | --------------- |
| asset          | string | 资产名称/缩写   |
| available      | string | 可用余额        |
| frozen_balance | string | 冻结余额        |
| total_balance  | string | 总额，冻结+余额 |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/account/one?asset=USDT
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 2068804b1a7f4d7fd5b3830b3bd6e1135a90565b20888f9f7c6b04596ff4c64e
ACCESS-TIMESTAMP: 2021-01-07T08:43:57.794Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:43:57.794ZGET/api/v3/spot/account/one?asset=USDT

Response:
{
    "code":200,
    "data":{
        "asset":"USDT",
        "available":"1389259.41398473",
        "frozen_balance":"30712.71076533",
        "total_balance":"1419972.12475007"
    }
}
```



### 私有接口-下单

```
按用户输入进行下单操作
限速规则：5次/1秒
HTTP POST /api/v3/spot/order
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                                |
| ------------- | ------ | -------- | ----------------------------------- |
| instrument_id | string | 是       | 币对名称，如BTC/USDT，用"/"分割     |
| direction     | string | 是       | 方向，direction=1:买 direction=2:卖 |
| price         | string | 是       | 下单价格                            |
| quantity      | string | 是       | 委托数量                            |

返回字段说明：

| 名称     | 类型   | 说明         |
| -------- | ------ | ------------ |
| order_id | string | 生成的订单id |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/order
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: c105a1e0302876df5fe7ba2d58ad469ac3258c9eb027acaa9101c583ae6a01bc
ACCESS-TIMESTAMP: 2021-01-07T08:51:22.580Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: {"instrument_id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}
preHash: 2021-01-07T08:51:22.580ZPOST/api/v3/spot/order{"instrument_id":"BTC/USDT","price":"37994.13","quantity":"1","direction":"2"}


Response:
{
    "code":200,
    "data":{
        "order_id":"2120223552677564416"
    }
}
```

### 私有接口-批量下单

```
按用户输入进行批量下单操作，只支持限价单
限速规则：3次/1秒
HTTP POST /api/v3/spot/batch_order
```

请求参数是一个数组对象,包含下面参数

| 名称          | 类型   | 是否必填 | 说明                                         |
| ------------- | ------ | -------- | -------------------------------------------- |
| instrument_id | string | 是       | 币对名称，如BTC/USDT，用"/"分割              |
| direction     | string | 是       | 下单方向 direction=1表示买 direction=2表示卖 |
| price         | string | 是       | 下单价格                                     |
| quantity      | string | 是       | 委托数量                                     |

返回字段说明：

| 名称     | 类型   | 说明         |
| -------- | ------ | ------------ |
| order_id | string | 生成的订单id |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/batch_order
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 78ad005f468b964ff4371ccb21ed7c37afc01b1dc335f49f4d9b704012ab63a8
ACCESS-TIMESTAMP: 2021-01-07T08:54:03.710Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: [{"instrument_id":"BTC/USDT","price":"37994.13","quantity":"2","direction":"2"},{"instrument_id":"BTC/USDT","price":"38994.13","quantity":"1","direction":"1"}]
preHash: 2021-01-07T08:54:03.710ZPOST/api/v3/spot/batch_order[{"instrument_id":"BTC/USDT","price":"37994.13","quantity":"2","direction":"2"},{"instrument_id":"BTC/USDT","price":"38994.13","quantity":"1","direction":"1"}]

Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2120224216451338240",
            "code":"200",
            "message":""
        },
        {
            "order_id":"2120224216547807232",
            "code":"200",
            "message":""
        }
    ]
}
```

### 私有接口-查询当前委托挂单列表

```
按用户请求进行当前委托挂单列表查询，订单未成交或部分成交,都在当前委托列表
限速规则：3次/1秒
HTTP GET /api/v3/spot/open_orders?instrument_id=BTC%2FUSDT&latestOrderId=
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                                                         |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_id | string | 是       | 币对名称，如BTC/USDT                                         |
| latestOrderId | string | 否       | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据 |

```
说明：
分页查询，每页返回20条
```

返回字段说明：

| 名称            | 类型   | 说明                                                         |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | 订单Id                                                       |
| base_asset      | string | 交易货币，如BTC                                              |
| quote_asset     | string | 计价货币，如USDT                                             |
| direction       | string | 下单方向                                                     |
| quantity        | string | 订单数量                                                     |
| filled_quantity | string | 已成交数量                                                   |
| amount          | string | 订单金额                                                     |
| filled_amount   | string | 已成交金额                                                   |
| Trade_pair_name | string | 币对名称                                                     |
| price           | string | 下单价格                                                     |
| status          | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled |
| order_time      | string | 下单时间                                                     |
| fee             | string | 手续费                                                       |
| taker_fee_rate  | string | taker费率                                                    |
| taker_fee_rate  | string | maker费率                                                    |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/open_orders?instrument_id=BTC%2FUSDT&latestOrderId=
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 1956e0de1ba125dd64402809605ed1ec9cb7135bd067d3a2629a23e47144867f
ACCESS-TIMESTAMP: 2021-01-07T09:09:18.301Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T09:09:18.301ZGET/api/v3/spot/open_orders?instrument_id=BTC%2FUSDT&latestOrderId=


Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2120224216451338240",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"sell",
            "quantity":"2",
            "filled_quantity":"0",
            "amount":"75988.26",
            "filled_amount":"0",
            "average_price":"",
            "status":"Open",
            "order_time":"2021-01-07T08:54:05.000Z",
            "update_time":"2021-01-07T08:54:05.000Z",
            "fee":"0",
            "taker_fee_rate":"0.013",
            "maker_fee_rate":"0.013",
            "trade_pair_name":"BTC/USDT",
            "price":"37994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2120223552677564416",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"sell",
            "quantity":"1",
            "filled_quantity":"0",
            "amount":"37994.13",
            "filled_amount":"0",
            "average_price":"",
            "status":"Open",
            "order_time":"2021-01-07T08:51:27.000Z",
            "update_time":"2021-01-07T08:51:26.000Z",
            "fee":"0",
            "taker_fee_rate":"0.013",
            "maker_fee_rate":"0.013",
            "trade_pair_name":"BTC/USDT",
            "price":"37994.13",
            "order_type":"limit"
        },
        ...
    ]
}
```

### 私有接口-查询历史委托单列表

```
按用户请求进行历史委托单列表查询，订单撤销或订单完全成交后, 才进入历史委托单列表
限速规则：3次/1秒
HTTP GET /api/v3/spot/closed_orders?instrument_id=BTC%2FUSDT&latestOrderId=
```

请求参数：

| 名称          | 类型   | 是否必填 | 说明                                                         |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| instrument_id | string | 是       | 币对名称，如BTC/USDT                                         |
| latestOrderId | string | 否       | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据 |

```
说明：
分页查询，每页返回20条
```

返回字段说明：

| 名称            | 类型   | 说明                                                         |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | 订单Id                                                       |
| base_asset      | string | 交易货币，如BTC                                              |
| quote_asset     | string | 计价货币，如USDT                                             |
| direction       | string | 下单方向                                                     |
| quantity        | string | 订单数量                                                     |
| filled_quantity | string | 已成交数量                                                   |
| amount          | string | 订单金额                                                     |
| filled_amount   | string | 已成交金额                                                   |
| Trade_pair_name | string | 币对名称                                                     |
| price           | string | 下单价格                                                     |
| status          | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled |
| order_time      | string | 下单时间                                                     |
| fee             | string | 手续费                                                       |
| taker_fee_rate  | string | taker费率                                                    |
| taker_fee_rate  | string | maker费率                                                    |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/closed_orders?instrument_id=BTC%2FUSDT&latestOrderId=
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 780778cf6a75c62fb38e88860902dec50340eb95d2512f738f4a51cdce620910
ACCESS-TIMESTAMP: 2021-01-07T09:13:33.807Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T09:13:33.807ZGET/api/v3/spot/closed_orders?instrument_id=BTC%2FUSDT&latestOrderId=

Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2120224216547807232",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"buy",
            "quantity":"1",
            "filled_quantity":"1",
            "amount":"38994.13",
            "filled_amount":"37284.246459",
            "average_price":"",
            "status":"Filled",
            "order_time":"2021-01-07T08:54:05.000Z",
            "update_time":"2021-01-07T08:54:05.000Z",
            "fee":"50.33373271965",
            "taker_fee_rate":"0.013",
            "maker_fee_rate":"0.013",
            "trade_pair_name":"BTC/USDT",
            "price":"38994.13",
            "order_type":"limit"
        },
        {
            "order_id":"2119464720976269312",
            "base_asset":"BTC",
            "quote_asset":"USDT",
            "direction":"sell",
            "quantity":"2",
            "filled_quantity":"2",
            "amount":"55988.26",
            "filled_amount":"61713.968507",
            "average_price":"",
            "status":"Filled",
            "order_time":"2021-01-05T06:36:07.000Z",
            "update_time":"2021-01-05T06:36:07.000Z",
            "fee":"83.31385748445",
            "taker_fee_rate":"0.013",
            "maker_fee_rate":"0.013",
            "trade_pair_name":"BTC/USDT",
            "price":"27994.13",
            "order_type":"limit"
        },
        ...
    ]
}
```

### 私有接口-查询指定订单信息

```
按用户请求进行指定订单查询，
限速规则：6次/1秒
HTTP GET /api/v3/spot/order_info?order_id=2120224216451338240
```

请求参数：

| 名称     | 类型   | 是否必填 | 说明   |
| -------- | ------ | -------- | ------ |
| order_id | string | 是       | 订单ID |

返回字段说明：

| 名称            | 类型   | 说明                                                         |
| --------------- | ------ | ------------------------------------------------------------ |
| order_id        | string | 订单Id                                                       |
| base_asset      | string | 交易货币，如BTC                                              |
| quote_asset     | string | 计价货币，如USDT                                             |
| direction       | string | 下单方向                                                     |
| quantity        | string | 订单数量                                                     |
| amount          | string | 订单金额                                                     |
| filled_amount   | string | 已成交金额                                                   |
| taker_fee_rate  | string | taker费率                                                    |
| maker_fee_rate  | string | maker费率                                                    |
| order_type      | string | 订单类型 limit表示限价单                                     |
| price           | string | 下单价格                                                     |
| status          | string | 订单状态，未成交：Open 完全成交：Filled 取消：Cancelled 部分成交：Partially cancelled |
| order_time      | string | 下单时间                                                     |
| fee             | string | 手续费                                                       |
| trade_pair_name | string | 币对名称                                                     |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/order_info?order_id=2120224216451338240
Method: GET
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: 91cd4bec3bcfe238c64a37f6c18fa7406a75b91ac3122991ce57d064deddc11f
ACCESS-TIMESTAMP: 2021-01-07T08:56:56.731Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: 
preHash: 2021-01-07T08:56:56.731ZGET/api/v3/spot/order_info?order_id=2120224216451338240

Response:
{
    "code":200,
    "data":{
        "order_id":"2120224216451338240",
        "base_asset":"BTC",
        "quote_asset":"USDT",
        "direction":"sell",
        "quantity":"2",
        "filled_quantity":"0",
        "amount":"75988.26",
        "filled_amount":"0",
        "status":"Open",
        "order_time":"2021-01-07T08:54:05.000Z",
        "fee":"0",
        "taker_fee_rate":"0.013",
        "maker_fee_rate":"0.013",
        "trade_pair_name":"BTC/USDT",
        "price":"37994.13",
        "order_type":"limit"
    }
}
```

### 私有接口-撤销指定委托单

```
按用户请求进行订单撤销，
限速规则：6次/1秒
HTTP POST /api/v3/spot/cancel_order
```

请求参数：

| 名称     | 类型   | 是否必填 | 说明     |
| -------- | ------ | -------- | -------- |
| order_id | string | 是       | 委托单ID |

返回字段说明：

| 名称     | 类型   | 说明         |
| -------- | ------ | ------------ |
| order_id | string | 撤销的订单Id |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/cancel_order
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: c860762f24679b3f68869ff6ce02cfddfabe178f5ee41aade9f00107cf880831
ACCESS-TIMESTAMP: 2021-01-07T09:17:49.591Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: {"order_id":"2119464720976269312"}
preHash: 2021-01-07T09:17:49.591ZPOST/api/v3/spot/cancel_order{"order_id":"2119464720976269312"}

Response:
{
    "code":200,
    "data":{
        "order_id":"2120223552677564416"
    }
}
```

### 私有接口-批量撤销委托单

```
按用户请求进行订单批量撤销，
限速规则：3次/1秒
HTTP POST /api/v3/spot/batch_cancel_order
```

请求参数：

| 名称     | 类型 | 是否必填 | 说明     |
| -------- | ---- | -------- | -------- |
| orderIds | list | 是       | 委托单ID |

返回字段说明：

| 名称     | 类型   | 说明         |
| -------- | ------ | ------------ |
| order_id | string | 撤销的订单Id |

```
Request:
Url: http://127.0.0.1:8604/api/v3/spot/batch_cancel_order
Method: POST
Headers: 
Accept: application/json
ACCESS-KEY: 47b4879d132d9160b0743e8d90abab25
ACCESS-SIGN: a6df76c43c41460f4dd5a3e7f4190a661d87eae70039c7571adbf8061b8de961
ACCESS-TIMESTAMP: 2021-01-07T09:19:56.090Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=en_US
Body: {"orderIds":["2120230657572687872","2120230657669156864"]}
preHash: 2021-01-07T09:19:56.090ZPOST/api/v3/spot/batch_cancel_order{"orderIds":["2120230657572687872","2120230657669156864"]}
		
Response:
{
    "code":200,
    "data":[
        {
            "order_id":"2120230657572687872",
            "code":"200",
            "message":""
        },
        {
            "order_id":"2120230657669156864",
            "code":"51800",
            "message":"The transaction has been traded, failure"
        }
    ]
}
```

## 错误代码汇总

| 错误代码 | message                                                      |
| -------- | ------------------------------------------------------------ |
| 429      | 请求太频繁                                                   |
| 430      | 该资产暂不支持API用户交易                                    |
| 10001    | "ACCESS_KEY"不能为空                                         |
| 10002    | "ACCESS_SIGN"不能为空                                        |
| 10003    | "ACCESS_TIMESTAMP"不能为空                                   |
| 10005    | 无效的ACCESS_TIMESTAMP                                       |
| 10006    | 无效的ACCESS_KEY                                             |
| 10007    | 无效的Content_Type，请使用“application/json”格式             |
| 10008    | 请求时间戳过期                                               |
| 10009    | 系统错误                                                     |
| 10010    | api 校验失败                                                 |
| 11000    | 必填参数不能为空                                             |
| 11001    | 参数值错误                                                   |
| 11002    | 参数值超过最大值限制                                         |
| 11003    | 第三方接口暂无数据返回                                       |
| 11004    | 订单价格精度不符合                                           |
| 11005    | 该币对暂未开通杠杆                                           |
| 11007    | 币对与资产不匹配                                             |
| 51800    | 已成交,撤单失败                                              |
| 51801    | 委托单不存在,撤单失败                                        |
| 51802    | 交易对不合法                                                 |
| 51803    | 买入价不可高于现价{0}%                                       |
| 51804    | 卖出价不可低于现价{0}%                                       |
| 51805    | 委托价最多小数位{0}                                          |
| 51806    | 委托量最多小数位{0}                                          |
| 51807    | 最少买入{0}                                                  |
| 51808    | 最少卖出{0}                                                  |
| 51809    | 余额不足或已冻结                                             |
| 51810    | 暂停卖出,恢复时间,以公告为准                                 |
| 51811    | 用户没有交易权限                                             |
| 51812    | 当前{0}小时周期的涨停价格为{1}，买入价格不能高于涨停价格，请调整买入价格 |
| 51813    | 当前{0}小时周期的跌停价格为{1}，卖出价格不能低于跌停价格，请调整卖出价格 |
| 51814    | 计划委托只能在未触发状态下撤单                               |
| 51815    | 委托类型错误                                                 |
| 51816    | 账户类型错误                                                 |
| 51817    | 交易对错误                                                   |
| 51818    | 交易方向错误                                                 |
| 51819    | 委托来源错误                                                 |
| 51820    | 触发价格错误                                                 |
| 51821    | 触发价格最多小数位{0}                                        |
| 51822    | 买入价格不可高于触发价{0}%                                   |
| 51823    | 卖出价格不可低于触发价{0}%                                   |
| 51824    | 委托价格错误                                                 |
| 51825    | 委托总额错误                                                 |
| 51826    | 委托总额最多小数位{0}                                        |
| 51827    | 委托量错误                                                   |
| 51828    | 高级委托挂单数量不能超过{0}                                  |
| 51829    | 触发价格应该高于最新成交价                                   |
| 51830    | 触发价格应该低于最新成交价                                   |
| 51831    | 限价错误                                                     |
| 51832    | 限价最多小数位{0}                                            |
| 51833    | 限价应该高于最新成交价                                       |
| 51834    | 限价应该低于最新成交价                                       |
| 51835    | 没有找到账户                                                 |
| 51836    | 委托不存在                                                   |
| 51837    | 委托订单号错误                                               |
| 51838    | 批量委托数量不能超过{0}                                      |
| 51839    | 账户冻结失败                                                 |
| 51840    | 查询账户失败                                                 |
| 51841    | 交易对未设置涨跌停                                           |
| 51843    | 查询涨跌停价格失败                                           |
| 51848    | 买入价格不可低于触发价{0}%                                   |
| 51849    | 卖出价格不可高于触发价{0}%                                   |
