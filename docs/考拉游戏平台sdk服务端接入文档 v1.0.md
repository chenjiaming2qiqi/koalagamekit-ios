# 考拉游戏平台sdk服务端接入文档 v1.0



> 此文档用于对接CP人员阅读。请确保按照文档要求来接入，方便各个人员处理；请确认当前对接游戏ID，对接请跟我们拿取paykey 固定对接入，签名验证参数用途，确保服务器安全过滤。



## 1. 登录接口

> `https://kaola.ijm6.com/Api/Member/CheckLogin`
>
> 用于用户在SDK端登录之后，CP方与我方服务端异步交互验证手段，确保用户合法性。

+ ### 参数说明

  | 字段名称 | 说明         | 是否必须 | 备注(区分大小写)                                         |
  | -------- | ------------ | -------- | -------------------------------------------------------- |
  | appid    | 游戏ID       | ✔        | 整形（如：123456），请与我方咨询当前对接游戏ID           |
  | uid      | 用户UID标识  | ✔        | 整形10位（如：123456789），SDK端登录会返回的值           |
  | token    | 用户令牌标识 | ✔        | varchar字符串类型32位，SDK端登录会返回对应gametoken的值  |
  | time     | 验证时间戳   | ✔        | 整形类型10位时间戳值, 混合验证签名， SDK端登录会返回的值 |
  | sessid   | 额外验证参数 | ✔        | Varchar 字符串类型30位，SDK端初始化返回的sessid对应值    |
  | sign     | 验签参       | ✔        | 签名。参见表下方的签名规则。                             |

  ```html
  签名规则：

  MD5(appid + uid + token + sessid + time + appkey) 转小写，其中+号为连接符，要按照顺序，不实际接入字符，appkey请与我们确认此固定值。
  ```

+ ### 请求回应说明

  CP方请求我方服务器后进行登录校验，我方接口响应输出字符串标识；

  其他值返回表示请求登录异常情况，不予许通过。

  | **响应字符串** | **备注说明**                 |
  | -------------- | ---------------------------- |
  | success        | 只有这一个值才算验证通过成功 |
  | fail_1         | token 不能空                 |
  | fail_2         | Sign签名错误                 |
  | fail_3         | 已经失时效                   |
  | fail_4         | 用户登录错误                 |
  | fail_5         | 非IP白，目前没有验证这个玩意 |





## 2. 充值发货接口

> 用户在SDK里面进行了付款充值，付款成功之后，由我方服务器通知CP服务器，进行虚拟游戏币发放， 请CP提供游戏发货地址接口。由POST方式通知。

+ ### 参数说明

  | 字段名称  | 说明         | 必须 | 备注(注意大小写)                                       |
  | --------- | ------------ | ---- | ------------------------------------------------------ |
  | orderId   | 我方订单号   | ✔    | 字符串类型50位之内                                     |
  | uid       | 我方UID标识  | ✔    | 整形类型10位，我方唯一用户标识                         |
  | amount    | 充值订单金额 | ✔    | 单位元，整形                                           |
  | serverId  | 充值服务器ID | ✔    | 字符类型50位之内，统一用区服ID标识                     |
  | extraInfo | 额外信息     | ✔    | 字符类型60位之内 ，CP调用SDK时传入的扩展参数，原样传回 |
  | test      | 测试标识     | ✔    | 整形类型，1为测试订单，其他值为正常订单                |
  | orderTime | 下单时间     | ✔    | 整形类型十位时间戳                                     |
  | billno    | 游戏方订单号 | ✔    | 字符串类型80位之内                                     |
  | sign      | 签名参数     | ✔    | 签名算法见表下方。                                     |

  ```html
  签名算法：

  MD5(orderId + uid + serverId + amount + extraInfo + orderTime + billno + test + paykey) 转小写，其中+号为连接符，不实际接入字符串，注意链接顺序哦，paykey请与我们确认此固定值。
  ```

  ​


+ ### 结果返回说明

  请求参数时候请返回对应的字符串给我们判断。

  | 返回值   | 说明                               |
  | -------- | ---------------------------------- |
  | success  | 成功标识                           |
  | exist    | 我方订单订单对多个CP订单，订单重复 |
  | wait     | 等待处理中                         |
  | user     | 不存在用户信息                     |
  | role     | 充值角色未找到                     |
  | serverid | 服务器错误                         |
  | sign     | 签名失败                           |
  | amount   | 充值金额验证错误                   |
  | ip       | IP白出错,我方IP：106.14.250.138    |
  | fail     | 异常或者失败其他情况               |

+ ### 多次请求

  网络请求异常或处理失败时候，解决当CP处理异常，我方会进行多次回调重发请求，当订单请求返回 fail 时 我方将会在处理多次定时通知，通知次数：fail的订单会每隔5，10，30，60，120分钟，总共5次的重复请求通知，直到CP发货为准，请务必保证不能重复发放逻辑的。