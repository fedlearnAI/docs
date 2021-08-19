# 京东科技自研联邦学习（fedlearn）客户端接口文档


目前的客户端对外的接口分为2个部分，一部分用于与协调端交互，一部分用于客户端内部调用（接入接口自动化操作或者图形界面等）。

两个部分会做一些简单的隔离，与协调端（coordinator）的交互需要进行鉴权，内部接口可以自由调用。在实践中，需要在网关或者防火墙进行权限配置，防止外部访问者调用内部接口。

用于建模的协调端调用统一以  **/co/** 开头，内部接口统一以 **/local/** 开头，详见下文叙述。 

### 一、多方建模的接口

#### 0 通用字段 
为提高系统安全性，每个从协调端发送过来的请求都带有token字段，用于权限验证。

同时，每个请求的返回结果均由code、status和可选的body组成，code是状态标识码，status是长度不超过32个字符的标识信息。

body是具体的返回体。

具体见下表格

| 请求参数 | 名称   | 类型       | 含义     | 是否必传 | 长度     | 备注 |
| -------- | ------ | ------------------- | -------- | -------- | -------- | ---- |
|          | token  | String              | 鉴权     | Y        | 50       |      |
| **响应结果** | 名称   | 类型                | 含义     | 是否必传 | 长度     | 备注 |
|          | code   | int                 | 异常码   | Y        | 32位int  |      |
|          | data   | Map<String, String> | 返回结果 | N        |          |      |
|          | status | String              | 状态信息 | Y        | 32位字符 |      |



#### 1.1  读取元数据


|          | 协议   | HTTP                | 接口     | co/metadata | 请求类型 | POST |
| -------- | ------ | ------------------- | -------- | ----------- | -------- | ---- |
| 请求参数 | 名称   | 类型                | 含义     | 是否必传    | 长度     | 备注 |
|          | token  | String              | 鉴权码   | Y           | 50       |      |
| 响应结果 | 名称   | 类型                | 含义     | 是否必传    | 备注     |      |
|          | code   | int                 | 异常码   | Y           |          |      |
|          | data   | Map<String, Object> | 返回结果 | N           |          |      |
|          | status | String              | 状态信息 | Y           |          |      |


请求示例：

```json
{
  "token": "nlp-77787"
}
```


成功返回示例

```json
{
  "status": "success",
  "code": 0,
  "data":{
    
  }
}
```

失败返回

```json
{
  "status": "fail",
  "code": -1
}
```



#### 1.2  ID对齐接口


| |  协议 | HTTP  |  接口 | co/match |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| token	| String | 鉴权码 |Y|20| |
||matchId	|String	|唯一id	|Y	|30||
||matchType	|MappingType	|对齐算法	|Y	|30||
||dataset	|String	|数据集	|Y	|30||
||phase	|int	|阶段	|Y	|30||
||body	|String	|请求体	|Y	|30||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|Map<String, String>|	返回结果	|N|	||
||status	|String|	状态信息|Y|	||


请求示例：

```json
{
  "token": "nlp-77787",
  "matchId": "666666",
  "matchType":"Vertical-md5",
  "dataset":"aa.csv"
}
```


创建成功时，返回
```json
{
    "status": "success",
    "code": 0
}
```

用户已存在时，返回
```json
{
    "status": "user exist",
    "code": -2
}
```

其他失败时，返回
```json
{
    "status": "fail",
    "code": -1
}
```



#### 1.3 训练接口

|metal |  协议 | HTTP  |  接口 | co/train/run |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| token	| String | |Y|20| |
||password	|String	|密码	|Y	|30||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||


请求示例：	
```json
{
    "username": "nlp",
    "password": "666666"
}
```
登录成功时，返回
```json
{
    "status": "success",
    "code": 0
}
```
用户不存在时
```json
{
    "status": "account not exist",
    "code": -1
}
```

其他错误返回:
```json
{
    "status": "fail",
    "code": -2
}
```
判断返回状态应该使用code 判断，0 是正常，负数是报错，正整数代表存在不影响结果的异常，主要用于部分特殊情况，下同



#### 1.4 训练查询

|metal |  协议 | HTTP  |  接口 | co/train/query |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| token	| String | |Y|20| |
||taskName	|String	|任务名称	|Y	|25||
||clientInfo | 复合类型 |客户端信息||||
||features | 复合类型 |特征信息     ||||
|响应结果|	名称	|类型	|含义	|是否必传||备注|
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|Y|  |返回生成的taskId|
||status	|String|	状态码|Y|	||

ClientInfo详情：
|名称|类型|含义|是否必传|长度|备注|
|-|-|-|-|-|-|
|ip|String|IP地址| Y | 20|
|port|String|端口|Y  |10|
|protocol|String|协议|Y|20|

Features详情：
|名称|类型|含义|是否必传|长度|备注|
|-|-|-|-|-|-|
|name|String|特征|Y	|20||
|dtype	|String|	类型|	Y|	8||
|describe|string|描述	|N|	40|特征信息|

请求示例：
```json
{
    "username": "att",
    "taskName": "训练",
    "dataset": "cl0_train.csv",
    "clientInfo": {
        "ip": "127.0.0.1",
        "port": 8081,
        "protocol": "http"
    },
    "features": [
        {
            "name": "年龄",
            "dtype": "int",
            "describe": ""
        },
        {
            "name": "性别",
            "dtype": "bool",
            "describe": ""
        }
    ]
}
```
成功返回示例：
```json
{
    "status":"success",
    "code":0,
    "data": {
        "taskId": 100
    }
}
```

#### 1.5 取回数据

|metal |  协议 | HTTP  |  接口 | co/train/receicve |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||username	| String | |Y|20| |
||taskId	|String	|任务名称	|Y	|10||
||clientInfo|      |||||
||features  ||||||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||


ClientInfo详情：
|名称|类型|含义|是否必传|长度|备注|
|-|-|-|-|-|-|
|ip|String|IP地址| Y | 20|
|port|String|端口|Y  |10|
|protocol|String|协议|Y|20|

Features详情：
|名称|类型|含义|是否必传|长度|备注|
|-|-|-|-|-|-|
|name|String|特征|Y	|20||
|dtype	|String|	类型|	Y|	8|
|describe|String|描述	|N|	40|特征信息|
|alignment|复合类型| 对齐信息|N|


**请求示例**：
```json
{
    "username": "abc",
    "taskId": 2,
    "taskName": "训练2",
    "dataset": "cl0_train.csv",
    "clientInfo": {
        "ip": "127.0.0.1",
        "port": 3333,
        "protocol": "http"
    },
    "features": [
        {
            "name": "age",
            "dtype": "int",
            "describe": "",
            "alignment": {
                "participant": "A",
                "feature": "age1"
            }
        },
        {
            "name": "gender",
            "dtype": "bool",
            "describe": "",
            "alignment": {
                "participant": "A",
                "feature": "sex"
            }
        }
    ]
}
```
**返回示例**：
```json
{
    "status":"success",
    "code":0,
    "data": {}
}
```


#### 1.6 推理
  分为三个部分, 已创建的任务，已加入的任务，和待加入（既不是自己创建，也未加入）的任务，通过category 参数区分

|metal |  协议 | HTTP  |  接口 | co/inference/start |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String |用户名 |Y|20| |
||category	|String	|任务名称	|Y	|25|已创建任务：created <br>已加入任务：joined <br>待加入任务：option|
|响应结果|	名称	|类型	|含义	|是否必传||备注|
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
  "username": "att",
  "category": "created"
}
```
返回示例：
```json
{
    "status":"success",
    "code":0,
    "data":{
        "taskList":[
            {
                "owner":"nlp_test",
                "taskName":"abcd",
                "taskId":5,
                "participants":[]
            },
            {
                "owner":"nlp_test",
                "taskName":"test",
                "taskId":4,
                "participants":[
                    "nlp_test1"
                ]
            },
            {
                "owner":"nlp_test",
                "taskName":"test3",
                "taskId":3,
                "participants":[
                    "nlp_test1"
                ]
            }
        ]
    }
}
```

#### 1.7 取回远端推理数据

|metal |  协议 | HTTP  |  接口 | co/inference/fetch |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||taskId	|String	|任务名称	|Y	|10||
||clientInfo |      |||||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "username":"nlp",
    "taskId": "101"
}
```

返回示例：
```json
{
    "status": "success",
    "code": 0,
    "data": {
        "task": {
            "taskId": 1,
            "taskName": "aaa",
            "taskOwner": "nlp",
            "participants": [
                "jd",
                "jdd",
                "jddd"
            ],
            "clientList": [
                {
                "ip": "10.222.113.150",
                "port": 8096,
                "protocol": "http",
                "token": "dhfwioehow3243",
                "dataset": "cl0_train.csv",
                "hasLabel":false
                },
                {
                    "ip": "10.222.113.150",
                    "port": 8096,
                    "protocol": "http",
                    "token": "dhfwioehow3243",
                    "dataset": "cl0_train.csv",
                    "hasLabel":false
                }
            ],
            "featureList": [
                {
                    "task_id": 100,
                    "username": "nlp",
                    "feature": "x0",
                    "feature_type": "float",
                    "feature_describe": "",
                    "dep_user": null,
                    "dep_feature": null
                },
                {
                    "task_id": 100,
                    "username": "nlp",
                    "feature": "x1",
                    "feature_type": "float",
                    "feature_describe": "",
                    "dep_user": null,
                    "dep_feature": null
                }
            ]
        }
    }
}
```



#### 1.8 远端推理文件存储

|metal|协议 | HTTP |  接口 | /co/inference/push |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||taskId	|String	|任务名称	|Y	|10||
||matchAlgorithm |String|id匹配的算法|Y|40||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||


请求示例：
```json
{
    "username": "nlp",
    "taskId": 92,
    "matchAlgorithm": "MD5"
}
```

启动成功时返回：
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "matchToken": "92-SecureBoost-20200604201446"
    }
}
```

#### 1.9 ID对齐进度查询
|metal |  协议 | HTTP |接口| /co/prepare/match/progress |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||matchToken |String  |id匹配的算法|Y|40||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求类型	POST	
```json
{
    "username": "admin",
    "matchToken":"XXX-XXXXX"
}
```

进度查询成功时返回： （百分比范围为0-100整数）
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "percent":30,
        "describes":"正在对齐"
  }
}
```


### 二、本地接口
#### 2.1  查询配置项

|meta | 协议 | HTTP  |  接口 | /local/config/query |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||token	|String	|任务名称	|Y	|20|秘钥，避免其他人修改配置文件|
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||status	|String|	状态码|Y|	||
||data	|dict|	返回结果	|Y|	||


请求示例：
```json
{
  "token":"1A2b3c4D"
}
```

返回示例：
```json
{
    "code": 0,
    "status": "success",
    "data": {
      "config":[
        { 
          "key": "appName",
          "value": "fedlearn-client",
          "desc":"应用名",
        },
        {
          "key": "appNort",
          "value": 8094,
          "desc":"服务端口"
        },
        {
          "key": "logSettings",
          "value": "/tmp/logback.xml",
          "desc":""
        },
        {
          "key": "trainSources",
          "value": [
            {
          	  "type":"csv",
          	  "trainBase":"/tmp",
          	  "fileName":"client1_train.csv"
            },
            {
              "type":"csv",
              "trainBase":"/tmp",
              "fileName":"client1_train.csv",
              "dataset":"client1_train"
            }
          ],
          "desc":"训练数据源"
        },
        {
          "key":"testSources",
          "value": [],
          "desc":""
        },
        {
          "key":"inferenceSources",
          "value": [],
          "desc":""
        }
     ],
      "sourceOption":[
        {
          "type":"csv",
          "trainBase":"/tmp",
          "fileName":"example.csv"
        },
        {
          "type":"db",
          "username":"user",
          "password":"12",
          "driver":"example.csv",
          "url":"jdbc:mysql://127.0.0.1:3306/nlp?",
          "table":"user_click"
        },
        {
          "type":"http",
          "trainBase":"/tmp",
          "fileName":"example.csv"
        }
      ]
  }
}
```


#### 2.2 更新配置

|metal |  协议 | HTTP  |  接口 | /local/config/update |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||token	|String	|任务名称	|Y	|20|秘钥，避免其他人修改配置文件|
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||status	|String|	状态码|Y|	||
||data	|dict|	返回结果	|N|	||

请求示例：
```json
{
  "token":"1a2b3c4d",
  "config":[
    {
      "key":"app.name",
      "value":"fedlearn-client"
    },
    {
      "key":"app.port",
      "value":8094
    },
    {
       "key":"log.settings",
       "value":"/tmp/logback.xml",
    },
    {
      "key": "train.sources",
      "value": [
        {
          "type":"csv",
          "trainBase":"/tmp",
          "fileName":"client1_train.csv",
          "dataset":"client1_train"
        },
        {
          "type":"csv",
          "trainBase":"/tmp",
          "fileName":"client1_train.csv",
          "dataset":"client1_train"
        }
       ]
    },
    {
      "key":"test.sources",
      "value": []
    },
    {
      "key":"inference.sources",
      "value": []
    }
   ]
}
```

返回示例：更新成功时返回
```json
{
    "code": 0,
    "status": "success",
}
```

如果传入的数据格式不对，返回

```json
{
    "code": -1,
    "status": "app name not set correctly"
}
```



#### 2.3 关闭模型推理

|metal |  协议 | HTTP  |  接口 | /api/inference/remote |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||modelToken	|String	|模型key	|Y	|10||
||path | String  |文件路径|Y|||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例
```json
{
  "username": “nlp”,
  "modelToken": “96-SecureBoost-20200605115258”,
  "path": “/aaa/bbb”
}
```

返回示例
```json
{
    "code": 0,
    "data": {
          "inferenceId": “LinearRegression_Model2934729347273947298”
    },
    "status": "success"
}
```

#### 2.4 查询本地推理调用次数

|metal |  协议 | HTTP  |  接口 | /api/inference/progress |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||inferenceId	|String	|任务名称	|Y	|10||
||clientInfo |      |||||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "username": "nlp",
    "inferenceId":"LinearRegression_Model2934729347273947298"
}
```

返回进度条
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "percent":0 
    }
}
```
当进度条到达100%时，返回前20条示例
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "percent":100, 
        "content":[
            {"uid":11, "score":"10.263121046652351"},
            {"uid":12, "score":"10.263121046652351"},
            {"uid":13, "score":"10.263121046652351"}
        ]
    }
}
```

#### 2.5 推理日志查询

|metal |  协议 | HTTP  |  接口 | /api/inference/query/log |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| username	| String | |Y|20| |
||modelToken	|String	|模型token	|Y	|50||
||startTime|String   |||||
||endTime | String  |||||
||pageIndex | int |||||
||pageSize |int |||||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

String		Y	50	
startTime	String	查询时间开始节点	Y	50	
endTime	String	查询时间结束节点	Y	50	
pageIndex
Integer	当前页	Y	10	从1开始
pageSize
Integer	每页数量	Y	10	


请求示例：
```json
{
    "modelToken": "97-SecureBoost-20200822170951",
    "username": "ak",
    "startTime":"2020-12-03 00:00:00",
    "endTime":"2020-12-03 23:00:00",
    "pageIndex": "1",
    "pageSize": "20"
}
```
返回示例：	
```json
{
    "code": 0,
    "data": {
        "inferenceList": [
         {
           "inferenceId": "3cac2d4f-d4b6-40b0-bfcb-f9d01a6e68ae",
           "caller": "JDD",
           "startTime": "2020-08-26 16:15:37",
           "endTime": "2020-08-26 16:15:38",
           "inferenceResult": "success",
           "requestNum": 94,
           "responseNum": 53
          },
         {
          "inferenceId": "3cac2d4f-d4b6-40b0-bfcb-f9d01a6e68ae",
           "caller": "MOB",
           "startTime": "2020-08-26 16:15:37",
           "endTime": "2020-08-26 16:15:38",
           "inferenceResult": "success",
           "requestNum": 94,
           "responseNum": 53
           }
        ],
        "inferenceCount": 2
    },
    "status": "success"
}
```

#### 2.6 读取本地模型

| metal| 协议    | HTTP   | 接口     | /api/model/read | 请求类型 | POST |
| --- | ------ | ------ | -------- | --------------- | -------- | ---- |
|请求参数| 名称    | 类型   | 含义     | 是否必传        | 长度     | 备注 |
|      | username   | String |     | Y               | 20       |      |
|    | modelToken | String | 模型key  | Y           | 10       |      |
|响应结果| 名称    | 类型   | 含义     | 是否必传        | 备注     |      |
|     | code       | int    | 异常码   | Y    |          |      |
|     | data       | dict   | 返回结果 | Y         |          |      |
|     | status     | String | 状态码   | Y         |          |      |

请求示例

```json
{
  "username": “admin”,
  "modelToken": “96-SecureBoost-20200605115258”,
}
```

返回示例

```json
{
    "code": 0,
    "data": {
          "modelContent": “x=1,y=2”
    },
    "status": "success"
}
```

#### 2.7 传入模型

| metal| 协议  | HTTP   | 接口     | /api/model/write | 请求类型 | POST |
| -----| ---- | ----- | -------- | ---------------- | -------- | ---- |
|请求参数| 名称 | 类型   | 含义     | 是否必传         | 长度     | 备注   |
|      | username| String |      | Y        | 20       |                |
|      | modelToken|String | 模型key  | Y      | 10       |      |
|      | modelContent| byte[] | 模型内容 | Y    |          |      |
|响应结果| 名称 | 类型 | 含义  | 是否必传         | 备注     |      |
|      | code | int  | 异常码   | Y        |          |      |
|      | data | map   | 返回结果 | N        |          |      |
|      |status| String | 状态码   | Y     |        |      |

请求示例

```json
{
  "username": “nlp”,
  "modelToken": “96-SecureBoost-20200605115258”,
  "modelContent": “x=1,y=2”
}
```

返回示例

```json
{
    "code": 0,
    "status": "success"
}
```

#### 

### 附录 错误码

#### 1.
