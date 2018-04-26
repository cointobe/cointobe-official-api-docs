# cointobe-official-api-docs
Official Documentation for the CoinToBe APIs and Streams

# 介绍
######  
欢迎使用CoinToBe开发者文档。此文档提供了账户管理、行情查询、交易功能等相关API。API分为账户、交易和行情三类。账户和交易API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。行情API提供市场的行情数据，所有行情接口都是公开的。


# 概述
###### 
市场概况和基础信息

  ### 撮合引擎
  ###### 
  撮合系统撮合订单的优先级按照价格优于时间的优先级来撮合，优先撮合价格更有优势的订单。当价格一致时按照下单时间顺序撮合，先下单的先撮合。比如深度列表中目前有3笔挂单等待成交，分别为1: 14900美金买1个比特币，2: 15000美金买2个比特币，3: 14900美金买1.5个比特币。他们是按时间顺序1-2-3进入撮合系统的，根据价格优先，系统优先撮合订单2，根据时间优先，1跟3优先撮合1。所以系统撮合顺序是2-1-3。
  
  #### 成交价说明
  ######
   订单撮合时成交价将按maker挂单价格成交，而非taker吃单价格。
例如：A用户在10000美元挂了1个BTC的买单，然后B用户以8000美元的价格下了一个卖单，因为A用户的订单先进入撮合系统挂在深度列表上，所以以A的价格为准，最终这笔订单最终以10000美元成交。
#### 订单生命周期
###### 
   订单进入撮合引擎后是“未成交”状态；如果一个订单被撮合而全部成交，那么他会变成“已成交”状态；一个订单被撮合可能出现部分成交，那么他的状态会变成“部分成交”状态，并且继续留在撮合队列中进行撮合；一个留在撮合队伍中等待撮合的订单被撤销，那么他的状态会变成“已撤销”状态；发起撤消到完成撤消的过程中有一个过程状态“撤单中”；被撤消或者全部成交的订单将被从撮合队列中剔除。
   
#### 币币交易限价规则
###### 
   为了防止用户下错大单造成市场异常波动和个人资金损失，币币交易设置了FOK限价规则：如果用户在币币交易所下的市价单/限价单可以与当前买卖盘内订单直接成交，那么系统会判断成交深度对应的价格与同方向盘口价的偏差是否超出30%。如果超过，则此订单将被系统立即全数撤销，否则此订单正常进行撮合。
例如：某用户在XRP/BTC交易区下了100BTC的市价买单(此时卖一价为0.00012)，系统判断订单完成成交后最新成交价为0.0002。此时，(0.0002-0.00012)/0.00012=66.7%>30%，用户的这笔市价买单将被立即撤销，不会和买卖盘内订单进行撮合。

### 费用
#### 交易费用
###### 
采用maker-taker收费规则，为鼓励挂单，maker挂单成交的手续费会比taker吃单成交的手续费低。同时，为鼓励成交，CTB会针对成交量大的客户采取手续费返还政策，详情请联系平台客服。




# API
###### 
REST API提供账户管理、行情查询和交易功能。
同时提供了WebSocket API，订阅WebSocket可以获取行情数据的推送。

### 请求
######
所有请求基于Https协议，请求头信息中contentType 需要统一设置为:'application/x-www-form-urlencoded’。 

请求交互说明

1、请求参数：根据接口请求参数规定进行参数封装。

2、提交请求参数：将封装好的请求参数通过POST或GET等方式提交至服务器。

3、服务器响应：服务器首先对用户请求数据进行参数安全校验，通过校验后根据业务逻辑将响应数据以JSON格式返回给用户。

4、数据处理：对服务器响应数据进行处理。

#### 错误
######
除非特殊说明，错误请求都通过HTTP 4xx或者状态码进行返回，返回内容还将包含错误原因、参数信息。您的HTTP库应配置为非2xx请求提供消息主体，以便您可以从主体读取消息字段。
#### 常见错误码
######
400	Bad Request – Invalid request format

401	Unauthorized – Invalid API Key

403	Forbidden – You do not have access to the requested resource

404	Not Found

500	Internal Server Error – We had a problem with our server

#### 成功
######
HTTP状态码200表示成功响应，并可能包含内容。如果响应含有内容，则将显示在相应的返回内容里面。

### 分页
######
所有返回数据集的REST请求使用游标分页。游标分页允许在结果的当前页面之前和之后获取结果，并且非常适合于实时数据。像/ trades，/ fill，/ orders这样默认返回最新内容的请求。根据当前的返回结果，后续请求可以在他的基础之上指定请求数据的方向，可以请求在这之前的，也可以请求在这之后的数据。
before和after游标可通过响应头CB-BEFORE和CB-AFTER使用。在请求初始请求页面时，请求应使用这些游标值。
#### 参数
######
Parameter	Default	Description

before		Request page before (newer) this pagination id.

after		Request page after (older) this pagination id.

limit	100	Number of results per request. Maximum 100. (default 100)

#### 例子
######
GET /orders?before=2&limit=30

### 标准规范
#### 时间戳
######
除非另外指定，API中的所有时间戳均以微秒为单位返回，符合ISO 8601标准。请确保您可以解析ISO 8601格式。
##### 例子  2014-11-06T10:34:47.123456Z
#### 数字
######
为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。
整数（如交易编号和顺序）不加引号。
#### ID
######
除非另有说明，大多数标识符是UUID。当提出一个需要UUID的请求时，以下两个形式（有和没有破折号）都被接受。
132fb6ae-456b-4654-b4e0-d681ac05cea1 或者132fb6ae456b4654b4e0d681ac05cea1

我们也要支持用户下单时可以传一个符合我们规范的uuid给我们

### 访问限制
#### REST API
######
公共接口：我们通过IP限制公共接口的调用：每秒3个请求，每秒最多6个请求。
私人接口：我们通过用户ID限制私人接口的调用：每秒5个请求，每秒最多10个请求。

某些接口的特殊限制在具体的接口上注明
#### Websocket API 
######
Websocket API将每个命令类型（例如：NewOrderSingle，OrderCancelRequest）限制为每秒50条命令。
### 验证
######
#### 生成API Key
######
在对任何请求进行签名之前，您必须通过网站创建一个API key。创建key后，您将获得2个必须记住的信息：

API Key

Secret

API Key和Secret将由随机生成和提供。

#### 发起请求
######
所有REST请求都必须包含以下标题：

ACCESS-KEY api key作为一个字符串。

ACCESS-SIGN 使用base64编码签名（请参阅签名消息）。

ACCESS-TIMESTAMP 作为您的请求的时间戳。

ACCESS-PASSPHRASE 您在创建API密钥时指定的密码。

所有请求都应该含有application/json类型内容，并且是有效的JSON。
#### 签名
#####
该ACCESS-SIGN标头是通过使用在prehash字符串BASE64解码秘密密钥创建HMAC SHA256生成timestamp + method + requestPath + body（其中，+表示字符串连接）和BASE64编码的输出。时间戳值与CB-ACCESS-TIMESTAMP标题相同。
这body是请求主体字符串或省略，如果没有请求正文（通常为GET请求）。
本method应该是大写。
请记住，在将其用作HMAC的密钥之前，首先对64位字母数字密码字符串进行base64解码（结果为64个字节）。另外，在发送头部之前，对摘要输出进行base64编码。
######
用户提交的参数除sign外，都要参与签名。
首先，将待签名字符串要求按照参数名进行排序(首先比较所有参数名的第一个字母，按abcd顺序排列，若遇到相同首字母，则看第二个字母，以此类推)。
例如：对于如下的参数进行签名
string[] parameters={"api_key=c821db84-6fbd-11e4-a9e3-c86000d26d7c","symbol=btc_cny","type=buy","price=680","amount=1.0"}; 生成待签名的字符串 amount=1.0&api_key=c821db84-6fbd-11e4-a9e3-c86000d26d7c&price=680&symbol=btc_cny&type=buy
然后，将待签名字符串添加私钥参数生成最终待签名字符串。
例如：
amount=1.0&api_key=c821db84-6fbd-11e4-a9e3-c86000d26d7c&price=680&symbol=btc_cny&type=buy&secret_key=secretKey
注意，"&secret_key=secretKey"为签名必传参数。
最后，是利用32位MD5算法，对最终待签名字符串进行签名运算，从而得到签名结果字符串(该字符串赋值于参数sign)，MD5计算结果中字母全部大写。
#### 选择时间戳
#####
该ACCESS-TIMESTAMP头必须是从UTC的时间的Unix Epoch开始的秒数。十进制值是允许的。
您的时间戳必须在api服务时间的30秒内，否则您的请求将被视为过期并被拒绝。如果您认为服务器和API服务器之间存在较大的时间偏差，那么我们建议您使用时间点来查询API服务器时间。
######
签名时间和服务器时间差30秒的请求将被系统拒绝


# 账户API
######
管理主账户在交易账户(币币交易)

# 币币API
######
获取币币交易的行情信息，账户信息，订单操作，订单查询，账单明细查询
### 私有API
######
私有接口可用于订单管理和账户管理。每个私有请求必须使用规范的验证形式进行签名。
私有接口需要使用您的API  key进行验证。您可以在这里生成API key。

### 币币账户
#### 账户余额列表
######
获取币币交易账户余额列表，查询各币种的余额，冻结和可用情况
##### HTTP请求
######
GET /api/v1/spot/accounts
##### 返回参数
######
参数名	描述
account_id          账户ID
currency	账户币种
balance	  帐户中总的币数量
frozen	资金被占用     available	可用于提现或交易的资金

######
Field	Description

account_id	Account ID

currency	the currency of the account

balance	total funds in the account

Frozen	funds on hold (not available for use)

available	funds available to withdraw* or trade

##### 冻结说明
######
当你下单的时候，订单所需的资金将被冻结。这部分资金将不能被用于其他订单或者提现。资金将一直被冻结直至订单被成交或者取消。

### 订单

#### 下单
######
提供limit和market订单模式。只有当您的账户有足够的资金才能下单。一旦下订单，您的账户资金将在订单生命周期期间被冻结。什么币种以及多少资金被冻结取决于订单指定的类型和参数。

##### HTTP请求
######
POST /api/v1/spot/orders

##### 通用参数
######
参数	描述

client_oid	[可选]由您设置的订单ID来识别您的订单

type	[可选] limit，market（默认是limit）

side	buy or sell

Code	有效的交易币对,eg:ltc_btc

######
例子：

Param	Description

client_oid	[optional] Order ID selected by you to identify your order

type	[optional] limit, or market (default is limit)

side	buy or sell

Code	A valid product id

##### 限价单特殊参数
######
参数	描述

price	每个币的价格

size	买入或卖出的币的数量

######
Param	Description

price	Price per bitcoin

size	Amount of BTC to buy or sell

##### 返回参数
######
参数名	描述

orderId	订单ID

result     结果
###### 例子：
{
    “orderId”: “d0c5340b-6d6c-49d9-b567-48c4bfca13d2",
      "result": false
}

参数名	描述

orderId	订单ID

price     每个币的价格

size	订单包含的币的数量

Code   交易对ID

side   订单方向

type  订单类型

filledVolume    成交数量

status   订单状态

##### 解释说明
######

Code
必须与有效的交易对相匹配。交易对列表可通过/ products接口获得。


type
下订单时，您可以指定订单类型。您指定的订单类型将影响是否需要其他订单参数以及撮合引擎如何执行您的订单。如果type没有指定，订单将默认为一个limit订单。
限价单既是默认订单类型，也是基本订单类型。限价单需要指定一个price和size。该size是数字货币买入或卖出的数量，price表示每个数字货币的价格。限价单将按指定价格或更好的价格成交。根据市场条件，卖单可以按照指定价格或者更高价格来成交，买单可以按照指定价格或更低价格成交。如果不能立即成交，那么限价单将成为深度列表里的一部分，直到被另一个新进来的订单成交或被用户撤单。
市价单不同于限价单，因为它们不提供价格保证。然而，他确实提供了一种不必指定价格，而以固定数量的数字货币或计价货币进行购买或出售的方式。市价单下单后会立即撮合，而不会把他挂在深度列表上。市价单总是takers并承担费用。在下达市价单时，您可以指定计价货币或交易货币的数量。计价货币数量决定账户余额的使用量，交易币种数量决定交易币种的交易量。

price
价格必须以quote_increment计价单位为准。计价单位是价格的最小单位。对于BTC-USDT产品，最小单位为0.0001。低于0.0001将不被接受。市价单不需要。

size
数量必须大于base_min_size，不得大于base_max_size。数量可以是交易币种最小单位（BTC-USDT交易对的BTC）的任何倍数，其中包括satoshi单位。size表示买入或卖出BTC（或交易货币）的数量。

funds
金额字段可选用于市价单。指定时，表示有相应计价货币的产品要买入。例如，BTC-USD交易时市价单指定150USD表示将将花150 USD购买BTC（包括所有费用）。

frozen
对于限价买单，我们将冻结价格 x 数量 x (1 + 费率)计价货币的数量。对于卖单，我们将冻结你想卖的交易币种的数量。实际费用在交易时进行评估。如果您取消部分成交或未成交的订单，剩余资金将被解除冻结。
对于指定的金额的市价买单，这部分金额将被冻结。对于一个卖单，指定数量的交易币种将被冻结。


订单生命周期
HTTP请求将在订单被拒绝（资金不足，参数无效等）或下单成功（由匹配引擎接受）时作出响应。一个200响应表示该订单被接收并且进入撮合。进入撮合的订单可以部分或全部立即成交（取决于价格和市场条件）。部分成交的时候将把订单的剩余数量进行持续撮合。完全成交的订单将进入已成交状态。

鼓励监听市场数据流的用户使用client_oid字段以便在接受到的信息中标识对应的数据。

响应
一个成功接受的订单将被分配一个订单ID。成功接受的订单指的是撮合引擎已经接受的订单。

未成交的订单不会过期，并将保持撮合状态，直到被成交或取消。


#### 撤销订单
######
撤销之前下的订单。
如果订单在其生命周期内没有任何成交，则其记录可能被清除。这意味着订单详细信息将不可用GET /orders/<order-id>来获取。
##### HTTP请求
###### 
DELETE /api/v1/spot/orders/<orderId>
 
##### 查询参数
######
参数名		描述

Code	[可选的]	只取消对应交易对的订单

##### 返回值
######
返回值 status=200 表示成功


#### 批量撤销
######
撤销所有未完成订单。返回被撤销订单的列表。最多撤销50个订单。无法保证一定撤销成功。
##### HTTP请求
######
DELETE /api/v1/spot/orders
##### 查询参数
######
参数名		描述

Code	[可选的]	只取消对应交易对的订单

##### 返回值
######
返回值 status=200 表示成功

#### 获取历史列表
######
获取历史订单，状态为：已完成，已撤销，撤销中的订单，只获取前2天的这个请求支持分页。
##### HTTP请求
######
GET /api/v1/spot/orders
##### 查询参数
######
参数		描述

status		仅限这些状态的订单列表。通过all返回所有这些状态的订单

Code	[可选的]	只列出特定交易对的订单

status：等待成交、部分成交、撤单中、已成交、撤单

前三个在order表，后两个在finish表

###### 例子：
Param		Description

status		Limit list of orders to these statuses. Passing all returns orders of all statuses.

product_id	[optional]	Only list orders for a specific product

##### 返回参数
######
参数名	描述

orderId	订单ID

price      每个币的价格

size	订单包含的币的数量

Code 

side

type

createdDate

filledVolume

status:open|partially-filled|filled|cancel|canceled,#成交状态（未成交，部分成交,完全成交，撤单中，已撤单）
######

[
    {
        "orderId":2222,
        "price": "0.10000000",
        "size": "0.01000000",
        "code": "BTC-USD",
        "side": "buy",
        "type": “limit",
        “createdDate": "2016-12-08T20:02:28.53864Z",
        “filledVolume": "0.00000000",
       "status": "open"
    },
    {
      “orderId":11111",
      "size": "1.00000000",
      "code": "BTC-USD",
      "side": “sell",
      "type": "market",
      "createdDate": "2016-12-08T20:09:05.508883Z",
      “filledVolume": “1.00000000",
      "status": “filled"
     }
]

##### 解释说明
######
订单状态和结算：

订单完成后，将不再放在深度列表中。已完成和结算已状态之间有一个小窗口期。当所有的交易已经结算，并且剩下的冻结资金（如果有的话）已经被释放的时候，订单就被结算了。

轮询：

对于大批量交易，强烈建议您维护自己的未成交委托订单列表，并使用其中一个流数据市场数据馈送来更新。开始交易时，您应该轮询订单，以获取任何未成交委托单的当前状态。

executedValue 是累计成交的数量*价格。
未成交的订单可能会根据市场情况在你发起请求和服务器响应之间改变状态。

#### 获取订单
##### HTTP请求
##### 返回参数
##### 解释说明
 

### 交易对


### 时间











