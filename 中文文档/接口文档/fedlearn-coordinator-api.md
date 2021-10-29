# 京东数科自研联邦学习（fedlearn）API文档

- **版本号:0.9.6**

目前的服务端提供给前端的接口分为三个部分五个模块，分别是系统服务与查询，对齐与训练，联邦推理和隐私推理。

详见下文叙述。

### 一、系统服务与查询

#### 1.1 通用参数查询
|表头|协议|HTTP|接口| /api/system/parameter |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| 无	| N/A | |N/A|N/A| |
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

返回示例：
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "model": [
            "LogisticRegression",
            "LinearRegression",
            "Kernel"
        ],
        "match": [
          "MD5",
          "RSA",
          "DH"
        ]
    }
}
```


#### 4.2  数据集和特征查询

|表头 |  协议 | HTTP |  接口 | /api/system/dataset |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| url |String	|客户端地址	|Y	|80||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "url":"http://127.0.0.1:8094"
}
```
返回示例：
```json
{
    "code": 0,
    "data": {
        "list": [
            {
                "dataset": "reg0_train.csv",
                "features": [
                    {
                        "name": "uid",
                        "dtype": "float"
                    },
                    {
                        "name": "HouseAge",
                        "dtype": "float"
                    },
                    {
                        "name": "y",
                        "dtype": "float"
                    }
                ]
            },
            {
                "dataset": "class0_train.csv",
                "features": [
                    {
                        "name": "uid",
                        "dtype": "float"
                    },
                    {
                        "name": "job",
                        "dtype": "float"
                    },
                    {
                        "name": "poutcome",
                        "dtype": "float"
                    },
                    {
                        "name": "y",
                        "dtype": "float"
                    }
                ]
            }
        ]
    },
    "status": "success"
}
```



### 二、实体对齐（ID对齐）
#### 2.1 发起对齐

|表头|协议 | HTTP |  接口 | /api/match/start |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||taskId	|String	|任务id	|Y	|80||
||matchAlgorithm |String|id对齐算法|Y|40||
||clientList | 复合类型 |客户端信息|Y|||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||


请求示例：
```json
{
    "taskId": "92",
    "matchAlgorithm": "MD5",
    "clientList": [
        {
          "url":"http://127.0.0.2:8094",
          "index":"uid",
          "dataset": "cl0_train.csv"
        },
        {
          "url":"http://fl.jd.com/train1",
          "index":"uid",
          "dataset": "cl0_train.csv"
        }
    ]
}
```

启动成功时返回：
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "matchId": "92-MD5-20200604201446"
    }
}
```

#### 2.2 对齐进度查询
|表头 |  协议 | HTTP |接口| /api/match/progress |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||matchId |String  |id匹配的算法|Y|40||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
    "matchId":"92-MD5-210718154231"
}
```

进度查询成功时返回： （百分比范围为0-100整数）
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "percent":30,
        "describe":"正在对齐"
  }
}
```

#### 2.3 对齐详情查询
|表头 |  协议 | HTTP |接口| /api/match/detail |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||matchId |String  |id匹配的算法|Y|40||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
    "matchId":"92-MD5-210718154231"
}
```

进度查询成功时返回： （百分比范围为0-100整数）
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "taskId":30,
        "matchType":"MD5"
  }
}
```
#### 2.4 对齐列表查询
|表头 |  协议 | HTTP |接口| /api/match/list |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||taskList	|List	|任务名称	|Y	|10||
||type	|String	|任务状态	|N 不传时返回所有状态对齐任务	||[RUNNING,COMPLETE,FAIL]|
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
    "taskList":["3","4"],
    "type":"COMPLETE"
}
```

进度查询成功时返回：
```json
{
  "code": 0,
  "status": "success",
  "data": {
    "matchList": [
      {
        "matchId": "4-RSA-201203161908",
        "taskId": 4,
        "runningStatus": "RUNNING"
      },
      {
        "matchId": "3-MD5-201125144444",
        "taskId": 3,
        "runningStatus": "COMPLETE"
      },
      {
        "matchId": "3-MD5-201203105616",
        "taskId": 3,
        "runningStatus": "FAIL"
      }
    ]
  }
}
```

#### 2.5 删除对齐
|表头 |  协议 | HTTP |接口| /api/match/delete |请求类型 |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|	|
||matchId	|String	|对齐ID	|Y	|40||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
    "matchId":"14-RSA-20121239089"
}
```

删除成功时返回：
```json
{
  "code": 0,
  "status": "success"
}
```

删除失败时返回：
```json
{
  "code": -4,
  "status": "delete fail,..."
}
```

### 三. 训练过程  包括开始、停止、暂停、恢复训练和训练进度查询、训练详情查询等
#### 3.1 训练参数查询

|表头 |协议|HTTP|接口|/api/train/option/|请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||algorithmType	|String	|算法类型	|Y	|20||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
    "algorithmType": "LinearRegression"
}
```

返回结果示例：	
```json
{
    "code": 0,
    "data": {
        "algorithmParams":[
			{
                "field": "maxEpoch",
                "name": "maxEpoch",
                "describe": [
                    "1",
                    "1000"
                ],
                "type": "NUMS",
                "defaultValue": 30.0
            },
            {
                "field": "metricType",
                "name": "metricType",
                "describe": [
                    "G_L2NORM"
                ],
                "type": "MULTI",
                "defaultValue": "G_L2NORM"
            },
            {
                "field": "optimizer",
                "name": "optimizer",
                "describe": [
                    "BatchGD",
                    "NEWTON"
                ],
                "type": "STRING",
                "defaultValue": "BatchGD"
            }
      ]
    },
    "status": "success"
}
```
type 共有三种，枚举型STRING，数值型NUMS，多选型MULTI

#### 3.2  启动训练

|表头 |  协议 | HTTP  |  接口 | /api/train/start |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||taskId	|String	|任务id	|Y	|10||
||model	|String	|算法名称	|Y	|20||
||matchId	|String	|对齐id	|Y	|20||
||algorithmParams	|dict	|算法参数	|Y	|||
||clientList | 复合类型 |客户端信息|Y|||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

algorithmParams详情：

| 名称 | 类型 | 含义 | 是否必传 | 长度 | 备注 |
| ---- | ---- | ---- | -------- | ---- | ---- |
|field	|String|	参数名称	|Y	|10 | |
|value	|String|	参数值	|Y	|10 | |

clientList详情：

| 名称     | 类型     | 含义       | 是否必传 | 长度 | 备注 |
| -------- | -------- | ---------- | -------- | ---- | ---- |
| url      | String   | 客户端地址 | Y        | 10   |      |
| dataset  | String   | 数据集名称 | Y        | 10   |      |
| features | 复合类型 | 特征信息   | N        |      |      |

请求示例：
```json
{
  "taskId": "92",
  "model": "SecureBoost",
  "matchId": "92-MD5-210718154231",
  "algorithmParams": [
    {
      "field": "secretKey",
      "value": "rtretwery454"
    },
    {
      "field": "encryptionAlgorithm",
      "value": "RSA"
    },
    {
      "field": "trainStepLimit",
      "value": "1000"
    },
    {
      "field": "eval_metric",
      "value": [
        "rmse",
        "mape"
      ]
    },
    {
      "field": "crossValidation",
      "value": 1
    }
  ],
  "clientList": [
    {
      "url": "http://127.0.0.1:8096",
      "dataset": "cl0_train.csv",
      "features": {
        "featureList": [
          {
            "name": "uid",
            "type": "float",
            "frequency": 1
          },
          {
            "name": "job",
            "type": "float",
            "frequency": 1
          }
        ],
        "index": "uid"
      }
    },
    {
      "url": "http://127.0.0.1:8094",
      "dataset": "cl1_train.csv",
      "features": {
        "featureList": [
          {
            "name": "uid",
            "type": "float",
            "frequency": 1
          },
          {
            "name": "housing",
            "type": "float",
            "frequency": 1
          }
        ],
        "index": "uid"
      }
    }
  ]
}
```


启动或者进度查询成功时返回： 
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "modelToken": "92-RandomForest-20200604201446"
    }
}
```
启动失败时：
```json
{
    "status": "fail, illegal parameter",
    "code": -1
}
```

#### 3.3  自动训练

|表头 |  协议 | HTTP  |  接口 | /api/train/auto |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||taskId	|String	|任务id	|Y	|10||
||model	|String	|算法名称	|Y	|20||
||matchId	|String	|对齐id	|Y	|20||
||tuneType	|String	|调参类型可选[BOHB,SMAC]	|Y	|20||
||algorithmParams	|dict	|算法参数	|Y	|||
||clientList | 复合类型 |客户端信息|Y|||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

algorithmParams详情：

| 名称 | 类型 | 含义 | 是否必传 | 长度 | 备注 |
| ---- | ---- | ---- | -------- | ---- | ---- |
|field	|String|	参数名称	|Y	|10 | |
|range	|List|	参数范围	|Y	|30 | |

clientList详情：

| 名称     | 类型     | 含义       | 是否必传 | 长度 | 备注 |
| -------- | -------- | ---------- | -------- | ---- | ---- |
| url      | String   | 客户端地址 | Y        | 10   |      |
| dataset  | String   | 数据集名称 | Y        | 10   |      |
| features | 复合类型 | 特征信息   | N        |      |      |

请求示例：
```json
{
  "taskId": "92",
  "model": "SecureBoost",
  "matchId": "92-MD5-210718154231",
  "tuneType":"BOHB",
  "algorithmParams": [
    {
      "field": "secretKey",
      "range": [0,1]
    },
    {
      "field": "trainStepLimit",
      "range": ["1000","100","10"]
    },
    {
      "field": "eval_metric",
      "range": [
        "rmse",
        "mape"
      ]
    },
    {
      "field": "crossValidation",
      "range": [1,10]
    }
  ],
  "clientList": [
    {
      "url": "http://127.0.0.1:8096",
      "dataset": "cl0_train.csv",
      "features": {
        "featureList": [
          {
            "name": "uid",
            "type": "float",
            "frequency": 1
          },
          {
            "name": "job",
            "type": "float",
            "frequency": 1
          }
        ],
        "index": "uid"
      }
    },
    {
      "url": "http://127.0.0.1:8094",
      "dataset": "cl1_train.csv",
      "features": {
        "featureList": [
          {
            "name": "uid",
            "type": "float",
            "frequency": 1
          },
          {
            "name": "housing",
            "type": "float",
            "frequency": 1
          }
        ],
        "index": "uid"
      }
    }
  ]
}
```

启动或者进度查询成功时返回： 
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "modelToken": "92-RandomForest-20200604201446"
    }
}
```
启动失败时：
```json
{
    "status": "fail, illegal parameter",
    "code": -1
}
```

#### 3.4  单个训练进度和指标（包括训练完成和训练失败的任务也可以查询）

|表头 |  协议 | HTTP |  接口 | /api/train/status |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||modelToken |String	|模型id	|Y	|50||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "modelToken": "92-RandomForest-200604201446"
}
```
响应结果示例：
```json
{
  "code": 0,
  "status": "success",
  "data": {
    "percent": 11,
    "runningType": "running",
    "message": "ok",
    "trainMetrics": [
      {
        "name": "rmse",
        "metric": [
          {
            "x": 0,
            "y": 1234
          },
          {
            "x": 1,
            "y": 1001
          }
        ]
      },
      {
        "name": "mape",
        "metric": [
          {
            "x": 0,
            "y": 105.6
          },
          {
            "x": 0,
            "y": 101.3
          }
        ]
      },
      {
        "name": "confusion",
        "metric": [
          {
            "x": 0,
            "y": "[[5,2],[3,5]]"
          },
          {
            "x": 1,
            "y": "[[7,0],[1,7]]"
          }
        ]
      },
      {
        "name": "fetureImportance",
        "metric": [
          {
            "127.0.0.1:8094=1": 50.23
          },
          {
            "127.0.0.1:8095=3": 20.01
          },
          {
            "127.0.0.1:8094=2": 5.1
          },
          {
            "127.0.0.1:8095=1": 2.0
          }
        ]
      }
    ],
    "validationMetrics": [
      {
        "name": "rmse",
        "metric": [
          {
            "x": 0,
            "y": 1234
          },
          {
            "x": 1,
            "y": 1001
          }
        ]
      },
      {
        "name": "mape",
        "metric": [
          {
            "x": 0,
            "y": 105.6
          },
          {
            "x": 0,
            "y": 101.3
          }
        ]
      },
      {
        "name": "confusion",
        "metric": [
          {
            "x": 0,
            "y": "[[5,2],[3,5]]"
          },
          {
            "x": 1,
            "y": "[[7,0],[1,7]]"
          }
        ]
      }
    ]
  }
}
```

runningType 是枚举类型，共有 running, suspend, resume,  complete, stop, fail 等类型。
message用来表示详细的运行信息，比如报错时 用来表示详细的报错信息。
trainMetrics是训练指标统计，validationMetrics是验证指标统计。

#### 3.5 单个训练完整训练参数，用于任务恢复，重进入等

|表头 |  协议 | HTTP  |  接口 | /api/train/parameter |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
|| modelToken|String	|模型key	|Y	|50||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "modelToken": "92-SecureBoost-20200604201446"
}
```
返回示例：
```json
{
  "status": "success",
  "code": 0,
  "data": {
    "taskId":"126",
    "matchId":"126-MD5-12012",
    "algorithmParams": [
        {
        "field": "num_boost_round",
        "name": "树的个数",
        "value": 5,
        "describe": ["1", "100"],
        "type": "NUMS"
        }
    ],
    "startTime": "2020-07-03 00:00:00",
    "endTime":"2021-10-29 17:32:39",
    "algorithmType": "SecureBoost"
    }
}
```

#### 3.6 训练任务列表

|表头 |  协议 | HTTP  |  接口 | /api/train/list |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||taskList	|List	|任务名称	|Y	|10||
||type	|String	|任务状态	|N	||不传时返回所有状态训练任务|
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "taskList":["3","4"],
    "type":"COMPLETE"
}
```
注：请求需要传taskId，不传的情况下会报错。
响应结果示例：

```json
{
    "code": 0,
    "data": {
        "trainList": [
            {
                "modelToken": "4-SecureBoost-201203161908",
                "taskId": 4,
                "runningStatus":"RUNNING"
            },
            {
                "modelToken": "3-SecureBoost-201125144444",
                "taskId": 3,
                "runningStatus":"COMPLETE"
            },
            {
                "modelToken": "3-SecureBoost-201203105616",
                "taskId": 3,
                "runningStatus":"STOP"
            }
        ]
    },
    "status": "success"
}
```

#### 3.7 训练停止/暂停和恢复

|表头 |  协议 | HTTP  |  接口 | /api/train/change |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||modelToken	|String	|模型key	|Y	|120||
|| type	| String |状态变更类型，[stop/suspend/resume] |Y|20| |
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

```json
{
  "type": "stop",
  "modelToken": "92-SecureBoost-20200604201446"
}
```

暂停成功时：
```json
{
    "code": 0,
    "status": "success"
}
```
暂停未成功时，返回
```json
{
    "code": -1,
    "status": "suspend fail"
}
```
暂停或者查询失败时：
```json
{
    "status": "fail",
    "code": -1
}
```

#### 3.8 删除已完成的训练

|表头 |  协议 | HTTP  |  接口 | /api/train/delete |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||modelToken	|String	|模型token	|Y	|50||
|响应结果 |	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "modelToken": "97-SecureBoost-20200822170951"
}
```
返回示例：
```json
{
    "code": 0,
    "status": "success"
}
```

注：此处删除模型为数据库model_table里面将该模型标记为删除，不再支持推理，但是客户端本地并未删除该模型文件。


### 四、推理（包括模型查询，开始预测、进度查询等）
#### 4.1 调用接口推理（同步接口）

|表头 |  协议 | HTTP  |  接口 | /api/inference/batch |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||uid	|List<String>	|任务名称	|Y	|1000||
||modelToken	|String	|模型id	|	|||
||clientList | 复合类型 |客户端信息|Y|||
||secureMode | boolean |安全推理|N||不传时默认为普通推理模式|
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
  "uid": ["11","12","13"],
  "modelToken":"83-LinearRegression-210718154534",
  "secureMode":"false",
  "clientList":[
    {
      "url": "http://10.222.113.150:8096",
      "dataset": "cl0_train.csv"
    },
    {
      "url": "http://10.222.113.150:8099",
      "dataset": "cl0_train.csv"
    }
  ]
}
```

返回示例：
```json
{
    "code": 0,
    "status": "success",
    "data": {
        "predict":[
            {"uid":"11", "score":["10.263121046652351"]},
        	{"uid":"12", "score":["10.263121046652351"]},
        	{"uid":"13", "score":["10.263121046652351"]}
            ]
    }
}
```

#### 4.2 远端推理

|表头 |  协议 | HTTP  |  接口 | /api/inference/remote |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||modelToken	|String	|模型key	|Y	|50||
||path | String  |文件路径|Y|||
||clientList | 复合类型 |客户端信息|Y|||
||userAddress | String |文件owner信息|Y|||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
  "modelToken": "96-SecureBoost-20200605115258",
  "path": "/aaa/bbb.csv",
  "userAddress": "http://10.222.113.150:8099",
  "clientList":[
    {
      "url": "http://10.222.113.150:8096",
      "dataset": "cl0_train.csv"
    },
    {
      "url": "http://10.222.113.150:8099",
      "dataset": "cl0_train.csv"
    }
  ],

}
```

返回示例
```json
{
    "code": 0,
    "data": {
          "inferenceId": "LinearRegression_Model2934729347273947298"
    },
    "status": "success"
}
```

#### 4.3 远端推理进度查询

|表头 |  协议 | HTTP  |  接口 | /api/inference/progress |请求类型  |POST|
|-----|-----|-----|-----|-----|-----|-|
|请求参数|名称 |类型	|含义	|是否必传	|长度	|备注	|
||inferenceId	|String	|任务名称	|Y	|10||
|响应结果|	名称	|类型	|含义	|是否必传|备注||
||code|	int|	异常码	|Y	|||
||data	|dict|	返回结果	|N|	||
||status	|String|	状态码|Y|	||

请求示例：
```json
{
    "inferenceId":"83-FederatedGB-210720154220-85b3e5cf4d8f4a2db9d710b4eda1a8f1"
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
当进度条到达100%时，返回示例：
```json
{
  "code": 0,
  "data": {
    "path": "/aaa/bbb_result.txt",
    "inferenceId": "83-FederatedGB-210720154220-85b3e5cf4d8f4a2db9d710b4eda1a8f1",
    "startTime": 1626766984185,
    "endTime": 1626766985502,
    "predictInfo": "推理完成",
    "percent": "100"
  },
  "status": "success"
}
```


### 附录 错误码
#### 1

