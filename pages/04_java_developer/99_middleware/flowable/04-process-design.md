# 流程设计

## 4.1 BPMN 2.0 基础元素

### 开始事件

开始事件表示流程的起点。

#### 空开始事件

```xml
<startEvent id="start" name="开始"></startEvent>
```

#### 消息开始事件

```xml
<startEvent id="messageStart" name="消息开始">
    <messageEventDefinition messageRef="startMessage"></messageEventDefinition>
</startEvent>
```

#### 定时器开始事件

```xml
<startEvent id="timerStart" name="定时器开始">
    <timerEventDefinition>
        <timeCycle>R3/PT1H</timeCycle>
    </timerEventDefinition>
</startEvent>
```

#### 信号开始事件

```xml
<startEvent id="signalStart" name="信号开始">
    <signalEventDefinition signalRef="startSignal"></signalEventDefinition>
</startEvent>
```

### 结束事件

结束事件表示流程的终点。

#### 空结束事件

```xml
<endEvent id="end" name="结束"></endEvent>
```

#### 错误结束事件

```xml
<endEvent id="errorEnd" name="错误结束">
    <errorEventDefinition errorRef="businessError"></errorEventDefinition>
</endEvent>
```

#### 终止结束事件

```xml
<endEvent id="terminateEnd" name="终止结束">
    <terminateEventDefinition></terminateEventDefinition>
</endEvent>
```

### 用户任务

用户任务需要人工处理。

```xml
<userTask id="userTask" name="用户任务" 
          flowable:assignee="${assignee}"
          flowable:formKey="taskForm">
    <documentation>这是一个用户任务</documentation>
</userTask>
```

### 服务任务

服务任务自动执行 Java 代码或调用外部服务。

#### Java 类方式

```xml
<serviceTask id="serviceTask" name="服务任务"
             flowable:class="com.example.MyDelegate">
</serviceTask>
```

#### 表达式方式

```xml
<serviceTask id="serviceTask" name="服务任务"
             flowable:expression="${myBean.doSomething()}">
</serviceTask>
```

#### 委托表达式方式

```xml
<serviceTask id="serviceTask" name="服务任务"
             flowable:delegateExpression="${myDelegate}">
</serviceTask>
```

### 网关

#### 排他网关

排他网关根据条件选择一条路径。

```xml
<exclusiveGateway id="exclusiveGateway" name="排他网关"></exclusiveGateway>

<sequenceFlow id="flow1" sourceRef="exclusiveGateway" targetRef="task1">
    <conditionExpression xsi:type="tFormalExpression">
        <![CDATA[${amount <= 1000}]]>
    </conditionExpression>
</sequenceFlow>

<sequenceFlow id="flow2" sourceRef="exclusiveGateway" targetRef="task2">
    <conditionExpression xsi:type="tFormalExpression">
        <![CDATA[${amount > 1000}]]>
    </conditionExpression>
</sequenceFlow>
```

#### 并行网关

并行网关同时启动所有分支。

```xml
<parallelGateway id="fork" name="分支"></parallelGateway>
<parallelGateway id="join" name="汇合"></parallelGateway>

<sequenceFlow sourceRef="fork" targetRef="task1"></sequenceFlow>
<sequenceFlow sourceRef="fork" targetRef="task2"></sequenceFlow>
<sequenceFlow sourceRef="task1" targetRef="join"></sequenceFlow>
<sequenceFlow sourceRef="task2" targetRef="join"></sequenceFlow>
```

#### 包含网关

包含网关根据条件选择一个或多个路径。

```xml
<inclusiveGateway id="inclusiveGateway" name="包含网关"></inclusiveGateway>
```

#### 事件网关

事件网关根据事件类型选择路径。

```xml
<eventBasedGateway id="eventGateway" name="事件网关"></eventBasedGateway>
```

### 序列流

序列流连接两个元素。

#### 无条件序列流

```xml
<sequenceFlow id="flow" sourceRef="task1" targetRef="task2"></sequenceFlow>
```

#### 条件序列流

```xml
<sequenceFlow id="flow" sourceRef="gateway" targetRef="task">
    <conditionExpression xsi:type="tFormalExpression">
        <![CDATA[${approved == true}]]>
    </conditionExpression>
</sequenceFlow>
```

#### 默认序列流

```xml
<exclusiveGateway id="gateway" default="defaultFlow"></exclusiveGateway>

<sequenceFlow id="defaultFlow" sourceRef="gateway" targetRef="defaultTask"></sequenceFlow>
```

## 4.2 常用流程模式

### 串行流程

```xml
<process id="serialProcess" name="串行流程">
    <startEvent id="start"></startEvent>
    <userTask id="task1" name="任务1" flowable:assignee="user1"></userTask>
    <userTask id="task2" name="任务2" flowable:assignee="user2"></userTask>
    <userTask id="task3" name="任务3" flowable:assignee="user3"></userTask>
    <endEvent id="end"></endEvent>
    
    <sequenceFlow sourceRef="start" targetRef="task1"></sequenceFlow>
    <sequenceFlow sourceRef="task1" targetRef="task2"></sequenceFlow>
    <sequenceFlow sourceRef="task2" targetRef="task3"></sequenceFlow>
    <sequenceFlow sourceRef="task3" targetRef="end"></sequenceFlow>
</process>
```

### 并行流程

```xml
<process id="parallelProcess" name="并行流程">
    <startEvent id="start"></startEvent>
    <parallelGateway id="fork" name="分支"></parallelGateway>
    <userTask id="task1" name="任务1" flowable:assignee="user1"></userTask>
    <userTask id="task2" name="任务2" flowable:assignee="user2"></userTask>
    <parallelGateway id="join" name="汇合"></parallelGateway>
    <endEvent id="end"></endEvent>
    
    <sequenceFlow sourceRef="start" targetRef="fork"></sequenceFlow>
    <sequenceFlow sourceRef="fork" targetRef="task1"></sequenceFlow>
    <sequenceFlow sourceRef="fork" targetRef="task2"></sequenceFlow>
    <sequenceFlow sourceRef="task1" targetRef="join"></sequenceFlow>
    <sequenceFlow sourceRef="task2" targetRef="join"></sequenceFlow>
    <sequenceFlow sourceRef="join" targetRef="end"></sequenceFlow>
</process>
```

### 分支流程

```xml
<process id="branchProcess" name="分支流程">
    <startEvent id="start"></startEvent>
    <userTask id="apply" name="申请" flowable:assignee="applicant"></userTask>
    <exclusiveGateway id="gateway" name="审批网关"></exclusiveGateway>
    <userTask id="managerApproval" name="经理审批" flowable:assignee="manager"></userTask>
    <userTask id="directorApproval" name="总监审批" flowable:assignee="director"></userTask>
    <endEvent id="end"></endEvent>
    
    <sequenceFlow sourceRef="start" targetRef="apply"></sequenceFlow>
    <sequenceFlow sourceRef="apply" targetRef="gateway"></sequenceFlow>
    <sequenceFlow sourceRef="gateway" targetRef="managerApproval">
        <conditionExpression xsi:type="tFormalExpression">
            <![CDATA[${amount <= 10000}]]>
        </conditionExpression>
    </sequenceFlow>
    <sequenceFlow sourceRef="gateway" targetRef="directorApproval">
        <conditionExpression xsi:type="tFormalExpression">
            <![CDATA[${amount > 10000}]]>
        </conditionExpression>
    </sequenceFlow>
    <sequenceFlow sourceRef="managerApproval" targetRef="end"></sequenceFlow>
    <sequenceFlow sourceRef="directorApproval" targetRef="end"></sequenceFlow>
</process>
```

### 会签流程

```xml
<process id="countersignProcess" name="会签流程">
    <startEvent id="start"></startEvent>
    <userTask id="countersignTask" name="会签任务">
        <multiInstanceLoopCharacteristics isSequential="false"
            flowable:collection="${approverList}"
            flowable:elementVariable="approver">
            <completionCondition>${nrOfCompletedInstances/nrOfInstances >= 0.6}</completionCondition>
        </multiInstanceLoopCharacteristics>
    </userTask>
    <endEvent id="end"></endEvent>
    
    <sequenceFlow sourceRef="start" targetRef="countersignTask"></sequenceFlow>
    <sequenceFlow sourceRef="countersignTask" targetRef="end"></sequenceFlow>
</process>
```

### 子流程

```xml
<process id="mainProcess" name="主流程">
    <startEvent id="start"></startEvent>
    <callActivity id="subProcess" name="子流程"
                  calledElement="subProcessDefinition"></callActivity>
    <endEvent id="end"></endEvent>
    
    <sequenceFlow sourceRef="start" targetRef="subProcess"></sequenceFlow>
    <sequenceFlow sourceRef="subProcess" targetRef="end"></sequenceFlow>
</process>

<process id="subProcessDefinition" name="子流程定义">
    <startEvent id="subStart"></startEvent>
    <userTask id="subTask" name="子任务"></userTask>
    <endEvent id="subEnd"></endEvent>
    
    <sequenceFlow sourceRef="subStart" targetRef="subTask"></sequenceFlow>
    <sequenceFlow sourceRef="subTask" targetRef="subEnd"></sequenceFlow>
</process>
```

## 4.3 流程设计工具

### Flowable Modeler

Flowable Modeler 是官方提供的 Web 流程设计器。

#### 集成 Flowable Modeler

```xml
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-ui-modeler-conf</artifactId>
    <version>${flowable.version}</version>
</dependency>

<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-ui-modeler-logic</artifactId>
    <version>${flowable.version}</version>
</dependency>

<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-ui-modeler-rest</artifactId>
    <version>${flowable.version}</version>
</dependency>
```

#### 配置 Modeler

```java
@Configuration
public class ModelerConfiguration {

    @Bean
    public ServletRegistrationBean modelerServlet() {
        ServletRegistrationBean registration = new ServletRegistrationBean(
            new ModelerServlet(), "/modeler/*"
        );
        registration.setLoadOnStartup(1);
        return registration;
    }
}
```

### Eclipse Designer

Eclipse Designer 是 Eclipse 插件，用于在 IDE 中设计流程。

#### 安装

1. 打开 Eclipse
2. Help → Install New Software
3. 添加更新站点：`http://flowable.org/designer/update/`
4. 选择 Flowable Designer 并安装

#### 使用

1. 新建 Flowable Project
2. 新建 BPMN Diagram
3. 拖拽元素设计流程
4. 导出为 BPMN XML 文件

### 在线设计器

#### 使用 bpmn.io

bpmn.io 是一个开源的 BPMN 设计器。

```html
<!DOCTYPE html>
<html>
<head>
    <title>BPMN 设计器</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bpmn-js@8.9.0/dist/assets/diagram-js.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bpmn-js@8.9.0/dist/assets/bpmn-font/css/bpmn.css">
</head>
<body>
    <div id="canvas" style="width: 100%; height: 600px;"></div>
    
    <script src="https://cdn.jsdelivr.net/npm/bpmn-js@8.9.0/dist/bpmn-viewer.development.js"></script>
    <script>
        var viewer = new BpmnJS({
            container: '#canvas'
        });
        
        viewer.importXML(bpmnXml, function(err) {
            if (!err) {
                viewer.get('canvas').zoom('fit-viewport');
            }
        });
    </script>
</body>
</html>
```

### 自定义设计器

如果需要更灵活的设计器，可以基于 bpmn.js 自定义开发。

#### 基础架构

```javascript
class CustomDesigner {
    constructor(containerId) {
        this.modeler = new BpmnJS({
            container: containerId,
            keyboard: {
                bindTo: window
            },
            propertiesPanel: {
                parent: '#properties-panel'
            },
            additionalModules: [
                propertiesPanelModule,
                propertiesProviderModule,
                customPropertiesProviderModule
            ],
            moddleExtensions: {
                flowable: flowableModdleDescriptor
            }
        });
    }

    async importXml(xml) {
        try {
            const result = await this.modeler.importXML(xml);
            this.modeler.get('canvas').zoom('fit-viewport');
            return result;
        } catch (err) {
            console.error('导入失败:', err);
        }
    }

    async exportXml() {
        try {
            const { xml } = await this.modeler.saveXML({ format: true });
            return xml;
        } catch (err) {
            console.error('导出失败:', err);
        }
    }
}
```

## 流程设计最佳实践

### 命名规范

- **流程定义 Key**: 使用小写字母和下划线，如 `leave_approval`
- **流程定义 Name**: 使用中文描述，如 `请假审批流程`
- **任务 ID**: 使用驼峰命名，如 `managerApproval`
- **任务 Name**: 使用中文描述，如 `经理审批`

### 流程结构

- 保持流程简洁，避免过度复杂
- 合理使用子流程拆分复杂逻辑
- 使用网关处理分支和并行
- 正确处理异常情况

### 性能考虑

- 避免过多的流程变量
- 合理设置历史记录级别
- 使用异步执行器处理耗时任务
- 定期清理历史数据

### 测试策略

- 单元测试每个流程节点
- 集成测试完整流程
- 测试各种边界情况
- 性能测试高并发场景
