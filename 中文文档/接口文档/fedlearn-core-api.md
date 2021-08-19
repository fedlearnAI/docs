# fedlearn 算法包函数接口

-----版本号：0.8.1

算法包对外API可以简单分为引用包、针对控制端的API和针对客户端的API，以下分别叙述。

## 一、引用 

在pom中引用fedlearn-core 

```xml
<dependency>
    <groupId>com.jdt.fedlearn</groupId>
    <artifactId>core</artifactId>
    <version>${fedlearn-core.version}</version>
</dependency>
```
或者下载jar文件引用

具体版本参照发布情况咨询开发者。

## 二、协调端函数接口
#### 0.需要外部输入的数据

部分信息，比如客户端信息，特征等 信息等需要训练前用户输入

```java
//我们将多方协商同意进行建模称为一个项目，一个项目中可以进行多次ID对齐，训练和推理等,
String projectId = "123";
//生成 ClientInfos， 以三方为例
ClientInfo[] clientInfos = new ClientInfo[3];
//根据实际情况构造客户端信息
clientInfos[0] = new ClientInfo("127.0.0.1", 80, "HTTP");
clientInfos[1] = new ClientInfo("127.0.0.1", 81, "HTTP", "/my/path");
clientInfos[2] = new ClientInfo("127.0.0.1", 82, "HTTP", "/my/path");

//特征信息, 与客户端信息一一对应, 每个Features都应该标注id列名称和label列字段名,
//如果该参与方的特征不包含label，可以不传入label或者传入null
Features[] features = new Features[3];
SingleFeature uid = new SingleFeature("uid", "int");
SingleFeature x0 = new SingleFeature("x0", "float");
SingleFeature x1 = new SingleFeature("x1", "int");
SingleFeature y = new SingleFeature("x1", "int");
List<SingleFeature> feature0 = Arrays.asList(uid, x0, x1, y);
features[0] = new Features(feature0, "uid", "y");
// 下面省略了特征构造过程
features[1] = new Features(new ArrayList(),"uid", null);
features[2] = new Features(new ArrayList(),"id");
```

#### 1.查询各项枚举值

- a.算法枚举，查询所有支持的算法

```java
import com.jdt.fedlearn.core.type.AlgorithmType;

//返回String类型的支持的算法
String[] algorithms = AlgorithmType.getAlgorithms();
//AlgorithmType 类型的
AlgorithmType[] algorithmTypes = AlgorithmType.getAlgorithmTypes();
```
- b.预处理方法枚举，查询所有支持的预处理
```java
import com.jdt.fedlearn.core.type.MappingType;

//返回String类型的支持的预处理方法
String[] algorithms = MappingType.getMappings();
//MappingType 类型的
MappingType[] algorithmTypes = MappingType.getMappingTypes();
```
#### 2.预处理

- a 构造预处理对象
```java
import com.jdt.fedlearn.core.preprocess.match.Prepare;
import com.jdt.fedlearn.core.type.MappingType;
import com.jdt.fedlearn.core.uniqueId.TokenUtil;
import com.jdt.fedlearn.core.entity.ClientInfo;
import com.jdt.fedlearn.core.preprocess.common.CommonPrepare;

MappingType mappingType = MappingType.VERTICAL_MD5;
Prepare prepare = CommonPrepare.construct(mappingType);
```
- b 预处理循环迭代
```java
import com.jdt.fedlearn.core.entity.ClientInfo;
import com.jdt.fedlearn.core.entity.common.CommonRequest;
import com.jdt.fedlearn.core.entity.common.CommonResponse;
import com.jdt.fedlearn.core.preprocess.common.MappingOutput;

List<CommonRequest> matchRequests = prepare.masterInit(clientInfos);
List<CommonResponse> matchResponses = new ArrayList<>();
while (prepare.isContinue()) {
  matchResponses = new ArrayList<>();
  //此处仅为示意，实际可采用并行请求实现
  for (CommonRequest request : matchRequests) {
    ClientInfo client = request.getClient();
    int p = request.getPhase();
    //消息传递函数需要自行实现
    String response = SendAndRecv.send(client, p, matchAlgorithm, request.getBody());
    matchResponses.add(new CommonResponse(client, response));
  }
  matchRequests = prepare.master(matchPhase, matchResponses);
}
//输出预处理过程的结果
MappingOutput mappingOutput = prepare.postMaster(matchResponses);
```

#### 3.参数处理

  - a.根据算法名称构造参数选项
```java
import com.jdt.fedlearn.core.parameter.common.CommonParameter;
import com.jdt.fedlearn.core.type.AlgorithmType;
import com.jdt.fedlearn.core.parameter.common.ParameterField;

//以联邦梯度提升树算法为例
AlgorithmType algorithmType = AlgorithmType.FederatedGB;
List<ParameterField> parameterList = CommonParameter.constructList(algorithmType);
```
  - b.根据用户选定的参数选项构造超参数类
```java
import com.jdd.fedlearn.core.type.AlgorithmType;

AlgorithmType algorithmType = AlgorithmType.FederatedGB;
//选择此处仅为示例，实际应该将用户在界面的选择完整传入 parameterMap
Map<String, Object> parameterMap = new HashMap<>();
parameterMap.put("numBoostRound", 5000);
//未指定的参数都会使用默认值
SuperParameter sp = CommonParameter.parseParameter(parameterMap, algorithmType);
```
#### 4.训练
  - a. 初始化
```java
import com.jdt.fedlearn.core.algorithm.common.CommonAlgorithm;
import com.jdt.fedlearn.core.algorithm.Algorithm;
import com.jdt.fedlearn.core.type.AlgorithmType;

import com.jdt.fedlearn.core.entity.ClientInfo;
import com.jdt.fedlearn.core.entity.feature.Features;

AlgorithmType algorithmType = AlgorithmType.FederatedGB;
//SuperParameter 来自于上个步骤构造的超参数 sp
Algorithm algorithm = CommonAlgorithm.algorithmConstruct(algorithmType, superParameter);

//来源于id对齐输出的结果
MappingOutput idMap = mappingOutput；
//根据用户选择的特征构造 featureList
Map<ClientInfo, Features> featureList = new HashMap<>();
Map<String, Object> other = new HashMap<>();
List<CommonRequest> initialRequests = algorithm.initControl(clientInfos, idMap, featureMap, other);
```
其中，clientInfos 是客户端列表

  - b 迭代
```java
List<CommonResponses> responses = new ArrayList<>();
List<CommonRequest> requests = algorithm.control(trainToken, phase, responses);
```
  - c 终止条件判断
```java
boolean isContinue = algorithm.isContinue();
```

  - d  指标统计
```java
Map<MetricType, List<Pair<Integer, Double>>> metric();
```
整个训练过程会组成一个自动机，demo 如下
```java
List<CommonRequest> requests = algorithm.initControl(clientInfos, idMap, featureMap, other);
while (algorithm.isContinue()) {
  List<CommonResponse> responses = new ArrayList<>();
  //保证给各客户端发送的状态一致
  for(CommonRequest req: requests){
  	response = SendAndRecv.send(req.getClient(), req.getPhase(), supportedAlgorithm, req.getBody());
    responses.add(response);
  }
   //更新请求
  requests = algorithm.control(responses);
  //读取参数
  Map<MetricType, List<Pair<Integer, Double>>> metrics = algorithm.metric();
}
```
#### 5.推理

推理过程分为初始化和正式运行两个阶段

- a 初始化
```java
// 初始化
String[] predictUid = new String["a", "b", "c"];
List<CommonRequest> requests = algorithm.initInference(clientInfos), predictUid);
```

- b 推理流程
```java
while (algorithm.isInferenceContinue()) {
    responses = new ArrayList<>();
    for (CommonRequest req : requests) {
        Message message = req.getBody();
        ClientInfo client = req.getClient();
        // 消息发送函数需自行实现
        CommonResponse response = send(client, req.getPhase(), message);
        responses.add(response);
    }
    requests = algorithm.inferenceControl(responses);
}
double res = algorithm.postInferenceControl(responses);
```


## 三、客户端函数接口
#### 0.需要外部输入的数据

与协调端类似，客户端也有需要外部输入的数据，比如从本地加载的数据，从接口输入的训练指示等

```java
//从数据源加载
String[][] rawData = new String[][0];
```

#### 1.客户端预处理过程

```java
import com.jdt.fedlearn.core.model.common.CommonModel;
import com.jdt.fedlearn.core.model.Model;

AlgorithmType algorithm = AlgorithmType.FederatedGB;
Model model = CommonModel.constructModel(algorithm);
```

#### 2.根据传入的算法名称构造 Model 类
```java
import com.jdt.fedlearn.core.model.common.CommonModel;
import com.jdt.fedlearn.core.model.Model;

AlgorithmType algorithm = AlgorithmType.FederatedGB;
Model model = CommonModel.constructModel(algorithm);
```
#### 3.客户端训练
- a 初始化
```java
import com.jdt.fedlearn.core.model.Model;
import com.jdt.fedlearn.core.entity.feature.Features;
import com.jdt.fedlearn.core.load.common.TrainData;

//以下参数从协调端传入
Mappingresult idMap;
SuperParameter sp;
Features features;
Map<String, Object> others;

TrainData trainData = model.trainInit(rawData, sp, idMap, features, others);
```

- b 训练过程
```java
//以下参数均来自于协调端
int phase;
String parameterData;
//trainData来自于上一步的初始化
TrainData trainData;
//返回结果根据
Message message = model.train(phase,  parameterData, trainData);
```
#### 4.客户端推理
- a 根据传入的二维数组和算法类型，构造InferenceData 类
```java
import com.jdt.fedlearn.core.load.common.CommonLoad;
import com.jdt.fedlearn.core.load.common.InferenceData;

AlgorithmType algorithm = AlgorithmType.FederatedGB;
InferenceData inferenceData = CommonLoad.constructInference(algorithm, sample);
```
- b.调用model inference 完成本地本轮推理
```java
// coordinator 传来的请求
Request req;
int phase = req.getPhase();
String parameterData = req.getParameterData();

Message  inference(phase, parameterData, data);
```
#### 5.模型序列化和反序列化
- a 序列化
```java
String  serializedModel = model.serialize();
```
- b.模型反序列化
```java
Model model;
model.deserialize(String serializedModel);
```

## 四、其他
算法包对外不抛异常，只返回错误码，调用方需关注
#### 1.异常处理

####2. 