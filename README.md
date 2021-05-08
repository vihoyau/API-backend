# API 后端文档汇总（内部）

**说明** ：该文档用于开发测试需要，如有需要协助或不规范的地方，请联系，为了您的阅读便利，我们会及时更正错误。感谢您的阅读。

## 1.远程更新数据库接口(update_data/index)

**微注**：参数说明如下，请使用正确签名及传参类型要求，暂时使用内网测试。

url:http://192.168.11.99:7002/update_data/index
<br/>
method: post

### 1.1请求参数，No.1.1表

| 字段名       | 变量名  |  必填 |  类型 |示例值|  描述|
| :--------  | :-----  | :----:  | :----:  | :----:  | :----:  |
| sql注入操作 | sql_type|是|boolean|1是、0否|进行sql操作,唯一,如果使用true则其他参数失效|
| 操作类型 | operation_type |是|string|update、select、insert|不支持delete,不需要则传"error"|
| sql语句 |sql_str |是|string|select * from table|如果不需要sql注入操作,则传"error"|
| 批量查询条件字段 |where_pro |是|string|{"OrderNo":"123","IMEI":"123"}|object->string json对象转成string后,再传,如果没有参数传"error"|
| 单个需要更新的json字段集 |update_obj |是|string|{"ICCID":"123"}|object->string json对象转成string后,再传,如果没有参数传"error"|
| 批量查询及更新 |where_update_arr |是|string|[{update_obj:{"ICCID":"123"},where_obj:{"OrderNo":"123","IMEI":"123"}}]|arr->string json对象转成string后,再传,如果没有参数传"error"|
| 随机数 | nonce |是|string|随机10位数|请求端需要对该值记录，避免重复请求|
| 加密密钥|api_key |是|string|不可见|签名密钥,请保管好|
| 当前时间戳 |timestamp |是|string|"1619764922000"|13位毫秒级别时间戳,默认凌晨3点更新,需要协商|
| 请求端ip | ip|是|string|"192.168.11.99"|记录更新请求端|
| 加密签名 |sign |是|string|16进制hex值|签名请看No.1.2表的规范|

### 1.2密签名参数，No.1.2表

**微注**：加密方式使用hmac(sha256)加密机制,优点是不可逆,防爆。加密需要做`encoding ='utf8'`转换
<br/>
步骤1：拼接字符串：str="nonce=123123&timestamp=123123&ip=192.168.11.99&api_key=123"
<br/>
步骤2：使用api_key加密成签名如下方法：
`crypto.createHmac('SHA256', api_key).update(str,encoding).digest('hex');`

| 字段名       | 变量名  |  必填 |  类型 |示例值|  描述|
| :--------  | :-----  | :----:  | :----:  | :----:  | :----:  |
| 随机数 | nonce |是|string|随机10位数|请求端需要对该值记录，避免重复请求|
| 加密密钥|api_key |是|string|不可见|签名密钥,请保管好|
| 当前时间戳 |timestamp |是|string|"1619764922000"|13位毫秒级别时间戳,默认凌晨3点更新,需要协商|
| 请求端ip | ip|是|string|"192.168.11.99"|记录更新请求端|

### 1.3返回事例及数据说明，No.1.3表

```json
{
	"code": 200,
	"msg": "更新成功、查询成功、插入成功",
	"data": [{
		"IMEI": "865998055241516"
	}]//具体返回数据，如果没有[]则证明查询结果没有

}```
```

| 字段名       | 变量名  |  必填 |  类型 |示例值|  描述|
| :--------  | :-----  | :----:  | :----:  | :----:  | :----:  |
| 返回码 | code |是|int|200、400、500|返回提示|
| 返回码200|200 |是|int||操作成功|
| 返回码400|400 |是|int||传递参数有误,请检查参数数据|
| 返回码500|500 |是|int||服务器报错,服务器有问题|
| 返回信息提醒|msg |是|string|sign签名有误||
| 返回数据结果|data |是|string||数据|

### 1.4后端处理逻辑
![程序设计](https://user-images.githubusercontent.com/30063579/116668639-33115e00-a9d0-11eb-8b4e-a3c6fc2b125b.png)
