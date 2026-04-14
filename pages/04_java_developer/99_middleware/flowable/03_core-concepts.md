# 核心概念

## 3.1 流程定义 (Process Definition)

### BPMN 2.0 规范

BPMN (Business Process Model and Notation) 是一种图形化表示业务流程的标准规范。Flowable 完全支持 BPMN 2.0 规范。

#### BPMN 2.0 核心元素

| 元素类型 | 符号 | 说明 |
|----------|------|------|
| 开始事件 | ![开始事件](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48Y2lyY2xlIGN4PSIyMCIgY3k9IjIwIiByPSIxOCIgZmlsbD0id2hpdGUiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSIyIi8+PC9zdmc+) | 流程的起点 |
| 结束事件 | ![结束事件](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48Y2lyY2xlIGN4PSIyMCIgY3k9IjIwIiByPSIxOCIgZmlsbD0id2hpdGUiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSI0Ii8+PC9zdmc+) | 流程的终点 |
| 用户任务 | ![用户任务](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA4MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iNzYiIGhlaWdodD0iMzYiIHg9IjIiIHk9IjIiIHJ4PSI0IiBmaWxsPSJ3aGl0ZSIgc3Ryb2tlPSIjMDAwIiBzdHJva2Utd2lkdGg9IjIiLz48L3N2Zz4=) | 需要人工处理的任务 |
| 服务任务 | ![服务任务](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA4MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iNzYiIGhlaWdodD0iMzYiIHg9IjIiIHk9IjIiIHJ4PSI0IiBmaWxsPSJ3aGl0ZSIgc3Ryb2tlPSIjMDAwIiBzdHJva2Utd2lkdGg9IjIiLz48Y2lyY2xlIGN4PSI0MCIgY3k9IjIwIiByPSI4IiBmaWxsPSJub25lIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMiIvPjwvc3ZnPg==) | 自动执行的任务 |
| 排他网关 | ![排他网关](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cG9seWdvbiBwb2ludHM9IjIwLDEgMzksMjAgMjAsMzkgMSwyMCIgZmlsbD0id2hpdGUiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSIyIi8+PC9zdmc+) | 条件分支 |
| 并行网关 | ![并行网关](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA0MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cG9seWdvbiBwb2ludHM9IjIwLDEgMzksMjAgMjAsMzkgMSwyMCIgZmlsbD0iI2ZmZiIgc3Ryb2tlPSIjMDAwIiBzdHJva2Utd2lkdGg9IjIiLz48bGluZSB4MT0iMTAiIHkxPSIyMCIgeDI9IjMwIiB5Mj0iMjAiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSIyIi8+PGxpbmUgeDE9IjIwIiB5MT0iMTAiIHgyPSIyMCIgeTI9IjMwIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMiIvPjwvc3ZnPg==) | 并行分支与汇合 |
| 序列流 | ![序列流](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHZpZXdCb3g9IjAgMCA4MCA0MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48bGluZSB4MT0iNSIgeTE9IjIwIiB4Mj0iNzUiIHkyPSIyMCIgc3Ryb2tlPSIjMDAwIiBzdHJva2Utd2lkdGg9IjIiLz48cG9seWdvbiBwb2ludHM9Ijc1LDIwIDY1LDE1IDY1LDI1IiBmaWxsPSIjMDAwIi8+PC9zdmc+) | 连接两个元素的箭头 |

### 流程定义部署

#### BPMN XML 示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:flowable="http://flowable.org/bpmn"
             targetNamespace="http://flowable.org/bpmn20">

    <process id="leaveApproval" name="请假审批流程" isExecutable="true">
        
        <!-- 开始事件 -->
        <startEvent id="start" name="开始"></startEvent>
        
        <!-- 用户任务 - 员工申请 -->
        <userTask id="apply" name="员工申请" flowable:assignee="${applyUser}">
            <documentation>员工填写请假申请</documentation>
        </userTask>
        
        <!-- 排他网关 - 经理审批 -->
        <exclusiveGateway id="managerGateway" name="经理审批"></exclusiveGateway>
        
        <!-- 用户任务 - 经理审批 -->
        <userTask id="managerApproval" name="经理审批" flowable:assignee="${manager}">
            <documentation>经理审批请假申请</documentation>
        </userTask>
        
        <!-- 服务任务 - 发送通知 -->
        <serviceTask id="sendNotification" name="发送通知" 
                     flowable:class="com.example.flowable.NotificationDelegate">
        </serviceTask>
        
        <!-- 结束事件 -->
        <endEvent id="end" name="结束"></endEvent>
        
        <!-- 序列流 -->
        <sequenceFlow id="flow1" sourceRef="start" targetRef="apply"></sequenceFlow>
        <sequenceFlow id="flow2" sourceRef="apply" targetRef="managerGateway"></sequenceFlow>
        <sequenceFlow id="flow3" sourceRef="managerGateway" targetRef="managerApproval">
            <conditionExpression xsi:type="tFormalExpression">
                <![CDATA[${days <= 3}]]>
            </conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="flow4" sourceRef="managerApproval" targetRef="sendNotification"></sequenceFlow>
        <sequenceFlow id="flow5" sourceRef="sendNotification" targetRef="end"></sequenceFlow>
        
    </process>

</definitions>
```

#### 部署流程定义

```java
import org.flowable.engine.RepositoryService;
import org.flowable.engine.repository.Deployment;
import org.flowable.engine.repository.ProcessDefinition;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ProcessDeploymentService {

    @Autowired
    private RepositoryService repositoryService;

    /**
     * 部署流程定义
     */
    public Deployment deployProcess() {
        Deployment deployment = repositoryService.createDeployment()
                .name("请假审批流程")
                .addClasspathResource("processes/leave-approval.bpmn20.xml")
                .deploy();
        
        return deployment;
    }

    /**
     * 查询流程定义
     */
    public List<ProcessDefinition> listProcessDefinitions() {
        return repositoryService.createProcessDefinitionQuery()
                .latestVersion()
                .orderByProcessDefinitionName()
                .asc()
                .list();
    }

    /**
     * 挂起流程定义
     */
    public void suspendProcessDefinition(String processDefinitionId) {
        repositoryService.suspendProcessDefinitionById(processDefinitionId);
    }

    /**
     * 激活流程定义
     */
    public void activateProcessDefinition(String processDefinitionId) {
        repositoryService.activateProcessDefinitionById(processDefinitionId);
    }

    /**
     * 删除流程定义
     */
    public void deleteProcessDefinition(String deploymentId) {
        repositoryService.deleteDeployment(deploymentId, true);
    }
}
```

### 版本管理

Flowable 自动管理流程定义的版本，每次部署都会创建一个新版本。

```java
// 查询所有版本的流程定义
List<ProcessDefinition> allVersions = repositoryService.createProcessDefinitionQuery()
        .processDefinitionKey("leaveApproval")
        .orderByProcessDefinitionVersion()
        .desc()
        .list();

// 获取最新版本
ProcessDefinition latestVersion = repositoryService.createProcessDefinitionQuery()
        .processDefinitionKey("leaveApproval")
        .latestVersion()
        .singleResult();

// 获取指定版本
ProcessDefinition specificVersion = repositoryService.createProcessDefinitionQuery()
        .processDefinitionKey("leaveApproval")
        .processDefinitionVersion(2)
        .singleResult();
```

## 3.2 流程实例 (Process Instance)

### 流程启动

```java
import org.flowable.engine.RuntimeService;
import org.flowable.engine.runtime.ProcessInstance;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class ProcessInstanceService {

    @Autowired
    private RuntimeService runtimeService;

    /**
     * 启动流程实例
     */
    public ProcessInstance startProcess(String processDefinitionKey, Map<String, Object> variables) {
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                processDefinitionKey,
                variables
        );
        return processInstance;
    }

    /**
     * 启动流程实例（带业务Key）
     */
    public ProcessInstance startProcessWithBusinessKey(
            String processDefinitionKey, 
            String businessKey, 
            Map<String, Object> variables) {
        
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                processDefinitionKey,
                businessKey,
                variables
        );
        return processInstance;
    }

    /**
     * 示例：启动请假流程
     */
    public ProcessInstance startLeaveProcess() {
        Map<String, Object> variables = new HashMap<>();
        variables.put("applyUser", "zhangsan");
        variables.put("manager", "lisi");
        variables.put("days", 2);
        variables.put("reason", "家里有事");

        return startProcess("leaveApproval", variables);
    }
}
```

### 流程变量

流程变量用于在流程实例中存储和传递数据。

#### 设置变量

```java
// 设置单个变量
runtimeService.setVariable(executionId, "key", value);

// 设置多个变量
Map<String, Object> variables = new HashMap<>();
variables.put("key1", value1);
variables.put("key2", value2);
runtimeService.setVariables(executionId, variables);

// 设置本地变量（仅在当前执行中有效）
runtimeService.setVariableLocal(executionId, "key", value);
```

#### 获取变量

```java
// 获取单个变量
Object value = runtimeService.getVariable(executionId, "key");

// 获取所有变量
Map<String, Object> variables = runtimeService.getVariables(executionId);

// 获取指定类型的变量
String stringValue = runtimeService.getVariable(executionId, "key", String.class);
Integer intValue = runtimeService.getVariable(executionId, "key", Integer.class);
```

#### 变量类型

Flowable 支持以下变量类型：

| 类型 | 说明 | 示例 |
|------|------|------|
| String | 字符串 | "hello" |
| Integer | 整数 | 100 |
| Long | 长整数 | 100000L |
| Double | 双精度浮点数 | 3.14 |
| Boolean | 布尔值 | true/false |
| Date | 日期 | new Date() |
| Serializable | 可序列化对象 | 自定义对象 |
| JSON | JSON 对象 | Jackson JsonNode |

### 流程状态

```java
/**
 * 查询流程实例
 */
public List<ProcessInstance> listActiveProcessInstances() {
    return runtimeService.createProcessInstanceQuery()
            .processDefinitionKey("leaveApproval")
            .active()
            .orderByProcessInstanceId()
            .desc()
            .list();
}

/**
 * 挂起流程实例
 */
public void suspendProcessInstance(String processInstanceId) {
    runtimeService.suspendProcessInstanceById(processInstanceId);
}

/**
 * 激活流程实例
 */
public void activateProcessInstance(String processInstanceId) {
    runtimeService.activateProcessInstanceById(processInstanceId);
}

/**
 * 删除流程实例
 */
public void deleteProcessInstance(String processInstanceId, String deleteReason) {
    runtimeService.deleteProcessInstance(processInstanceId, deleteReason);
}

/**
 * 触发信号
 */
public void triggerSignal(String signalName) {
    runtimeService.signalEventReceived(signalName);
}

/**
 * 触发消息
 */
public void triggerMessage(String messageName, String executionId) {
    runtimeService.messageEventReceived(messageName, executionId);
}
```

## 3.3 任务 (Task)

### 用户任务

用户任务是需要人工处理的任务。

#### 查询任务

```java
import org.flowable.engine.TaskService;
import org.flowable.task.api.Task;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TaskQueryService {

    @Autowired
    private TaskService taskService;

    /**
     * 查询待办任务
     */
    public List<Task> listTodoTasks(String assignee) {
        return taskService.createTaskQuery()
                .taskAssignee(assignee)
                .active()
                .orderByTaskCreateTime()
                .desc()
                .list();
    }

    /**
     * 查询候选任务
     */
    public List<Task> listCandidateTasks(String userId) {
        return taskService.createTaskQuery()
                .taskCandidateUser(userId)
                .active()
                .orderByTaskCreateTime()
                .desc()
                .list();
    }

    /**
     * 查询用户组任务
     */
    public List<Task> listGroupTasks(String groupId) {
        return taskService.createTaskQuery()
                .taskCandidateGroup(groupId)
                .active()
                .orderByTaskCreateTime()
                .desc()
                .list();
    }

    /**
     * 查询流程实例的任务
     */
    public List<Task> listTasksByProcessInstanceId(String processInstanceId) {
        return taskService.createTaskQuery()
                .processInstanceId(processInstanceId)
                .orderByTaskCreateTime()
                .desc()
                .list();
    }
}
```

#### 任务办理

```java
@Service
public class TaskCompleteService {

    @Autowired
    private TaskService taskService;

    /**
     * 完成任务
     */
    public void completeTask(String taskId) {
        taskService.complete(taskId);
    }

    /**
     * 完成任务（带变量）
     */
    public void completeTask(String taskId, Map<String, Object> variables) {
        taskService.complete(taskId, variables);
    }

    /**
     * 示例：完成请假审批任务
     */
    public void completeApprovalTask(String taskId, boolean approved) {
        Map<String, Object> variables = new HashMap<>();
        variables.put("approved", approved);
        variables.put("approvalComment", approved ? "同意" : "不同意");
        
        completeTask(taskId, variables);
    }
}
```

### 服务任务

服务任务是自动执行的任务，通常用于调用外部服务或执行 Java 代码。

#### Java 委托类

```java
import org.flowable.engine.delegate.DelegateExecution;
import org.flowable.engine.delegate.JavaDelegate;
import org.springframework.stereotype.Component;

@Component("notificationDelegate")
public class NotificationDelegate implements JavaDelegate {

    @Override
    public void execute(DelegateExecution execution) {
        // 获取流程变量
        String applyUser = (String) execution.getVariable("applyUser");
        String reason = (String) execution.getVariable("reason");
        
        // 发送通知
        System.out.println("发送通知给 " + applyUser);
        System.out.println("请假原因: " + reason);
        
        // 设置流程变量
        execution.setVariable("notificationSent", true);
    }
}
```

#### BPMN 配置

```xml
<serviceTask id="sendNotification" name="发送通知" 
             flowable:delegateExpression="${notificationDelegate}">
</serviceTask>
```

### 任务分配

#### 直接分配

```xml
<userTask id="task1" name="任务" flowable:assignee="zhangsan">
</userTask>
```

#### 表达式分配

```xml
<userTask id="task1" name="任务" flowable:assignee="${manager}">
</userTask>
```

#### 候选用户

```xml
<userTask id="task1" name="任务">
    <potentialOwner>
        <resourceAssignmentExpression>
            <formalExpression>user(zhangsan), user(lisi)</formalExpression>
        </resourceAssignmentExpression>
    </potentialOwner>
</userTask>
```

#### 候选组

```xml
<userTask id="task1" name="任务">
    <potentialOwner>
        <resourceAssignmentExpression>
            <formalExpression>group(managers)</formalExpression>
        </resourceAssignmentExpression>
    </potentialOwner>
</userTask>
```

#### 任务监听器分配

```java
import org.flowable.engine.delegate.TaskListener;
import org.flowable.task.service.delegate.DelegateTask;

public class AssignmentListener implements TaskListener {

    @Override
    public void notify(DelegateTask delegateTask) {
        // 根据业务逻辑分配任务
        String assignee = calculateAssignee(delegateTask);
        delegateTask.setAssignee(assignee);
    }

    private String calculateAssignee(DelegateTask delegateTask) {
        // 自定义分配逻辑
        return "zhangsan";
    }
}
```

### 任务委托

```java
@Service
public class TaskDelegateService {

    @Autowired
    private TaskService taskService;

    /**
     * 认领任务
     */
    public void claimTask(String taskId, String userId) {
        taskService.claim(taskId, userId);
    }

    /**
     * 取消认领
     */
    public void unclaimTask(String taskId) {
        taskService.unclaim(taskId);
    }

    /**
     * 委派任务
     */
    public void delegateTask(String taskId, String userId) {
        taskService.delegateTask(taskId, userId);
    }

    /**
     * 转办任务
     */
    public void assignTask(String taskId, String userId) {
        taskService.setAssignee(taskId, userId);
    }

    /**
     * 添加候选用户
     */
    public void addCandidateUser(String taskId, String userId) {
        taskService.addCandidateUser(taskId, userId);
    }

    /**
     * 添加候选组
     */
    public void addCandidateGroup(String taskId, String groupId) {
        taskService.addCandidateGroup(taskId, groupId);
    }
}
```

## 3.4 执行 (Execution)

### 执行对象

执行对象代表流程实例中的一个执行路径。

```java
import org.flowable.engine.RuntimeService;
import org.flowable.engine.runtime.Execution;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ExecutionService {

    @Autowired
    private RuntimeService runtimeService;

    /**
     * 查询执行对象
     */
    public List<Execution> listExecutionsByProcessInstanceId(String processInstanceId) {
        return runtimeService.createExecutionQuery()
                .processInstanceId(processInstanceId)
                .list();
    }

    /**
     * 查询活动的执行对象
     */
    public List<Execution> listActiveExecutions(String activityId) {
        return runtimeService.createExecutionQuery()
                .activityId(activityId)
                .list();
    }

    /**
     * 触发执行
     */
    public void triggerExecution(String executionId) {
        runtimeService.trigger(executionId);
    }

    /**
     * 触发执行（带变量）
     */
    public void triggerExecution(String executionId, Map<String, Object> variables) {
        runtimeService.trigger(executionId, variables);
    }
}
```

### 执行路径

流程实例可能有多个执行路径，特别是在并行流程中。

```java
/**
 * 获取流程实例的所有执行路径
 */
public void printExecutionTree(String processInstanceId) {
    List<Execution> executions = runtimeService.createExecutionQuery()
            .processInstanceId(processInstanceId)
            .list();
    
    for (Execution execution : executions) {
        System.out.println("Execution ID: " + execution.getId());
        System.out.println("Parent ID: " + execution.getParentId());
        System.out.println("Activity ID: " + execution.getActivityId());
        System.out.println("---");
    }
}
```

### 多实例

多实例用于处理需要多次执行的任务，如会签。

#### BPMN 配置

```xml
<userTask id="multiInstanceTask" name="多实例任务" isForCompensation="false">
    <multiInstanceLoopCharacteristics isSequential="false"
        flowable:collection="${assigneeList}"
        flowable:elementVariable="assignee">
        <completionCondition>${nrOfCompletedInstances/nrOfInstances >= 0.5}</completionCondition>
    </multiInstanceLoopCharacteristics>
</userTask>
```

#### 多实例变量

| 变量名 | 说明 |
|--------|------|
| nrOfInstances | 总实例数 |
| nrOfCompletedInstances | 已完成实例数 |
| nrOfActiveInstances | 活跃实例数 |
| loopCounter | 当前循环计数器 |

#### Java 代码

```java
/**
 * 启动多实例流程
 */
public ProcessInstance startMultiInstanceProcess() {
    Map<String, Object> variables = new HashMap<>();
    List<String> assigneeList = Arrays.asList("user1", "user2", "user3");
    variables.put("assigneeList", assigneeList);
    
    return runtimeService.startProcessInstanceByKey("multiInstanceProcess", variables);
}
```
