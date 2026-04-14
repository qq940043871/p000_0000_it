# API 使用

## 5.1 RepositoryService

RepositoryService 用于管理流程定义和部署。

### 流程定义管理

```java
import org.flowable.engine.RepositoryService;
import org.flowable.engine.repository.Deployment;
import org.flowable.engine.repository.ProcessDefinition;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.InputStream;
import java.util.List;
import java.util.zip.ZipInputStream;

@Service
public class RepositoryServiceExample {

    @Autowired
    private RepositoryService repositoryService;

    /**
     * 部署流程定义（单个文件）
     */
    public Deployment deployProcess(String name, String resourcePath) {
        return repositoryService.createDeployment()
                .name(name)
                .addClasspathResource(resourcePath)
                .deploy();
    }

    /**
     * 部署流程定义（ZIP文件）
     */
    public Deployment deployProcessFromZip(String name, InputStream zipInputStream) {
        ZipInputStream zip = new ZipInputStream(zipInputStream);
        return repositoryService.createDeployment()
                .name(name)
                .addZipInputStream(zip)
                .deploy();
    }

    /**
     * 部署流程定义（字符串）
     */
    public Deployment deployProcessFromString(String name, String bpmnXml) {
        return repositoryService.createDeployment()
                .name(name)
                .addString("process.bpmn20.xml", bpmnXml)
                .deploy();
    }

    /**
     * 查询所有流程定义
     */
    public List<ProcessDefinition> listAllProcessDefinitions() {
        return repositoryService.createProcessDefinitionQuery()
                .orderByProcessDefinitionName()
                .asc()
                .list();
    }

    /**
     * 查询最新版本的流程定义
     */
    public List<ProcessDefinition> listLatestProcessDefinitions() {
        return repositoryService.createProcessDefinitionQuery()
                .latestVersion()
                .orderByProcessDefinitionName()
                .asc()
                .list();
    }

    /**
     * 根据Key查询流程定义
     */
    public ProcessDefinition getProcessDefinitionByKey(String processDefinitionKey) {
        return repositoryService.createProcessDefinitionQuery()
                .processDefinitionKey(processDefinitionKey)
                .latestVersion()
                .singleResult();
    }

    /**
     * 挂起流程定义
     */
    public void suspendProcessDefinition(String processDefinitionId) {
        repositoryService.suspendProcessDefinitionById(processDefinitionId);
    }

    /**
     * 挂起流程定义（级联挂起流程实例）
     */
    public void suspendProcessDefinitionCascade(String processDefinitionId) {
        repositoryService.suspendProcessDefinitionById(processDefinitionId, true, null);
    }

    /**
     * 激活流程定义
     */
    public void activateProcessDefinition(String processDefinitionId) {
        repositoryService.activateProcessDefinitionById(processDefinitionId);
    }

    /**
     * 删除部署
     */
    public void deleteDeployment(String deploymentId) {
        repositoryService.deleteDeployment(deploymentId);
    }

    /**
     * 删除部署（级联删除流程实例）
     */
    public void deleteDeploymentCascade(String deploymentId) {
        repositoryService.deleteDeployment(deploymentId, true);
    }

    /**
     * 获取流程定义资源
     */
    public InputStream getProcessDefinitionResource(String processDefinitionId) {
        return repositoryService.getProcessModel(processDefinitionId);
    }

    /**
     * 获取流程定义图片
     */
    public InputStream getProcessDefinitionDiagram(String processDefinitionId) {
        return repositoryService.getProcessDiagram(processDefinitionId);
    }
}
```

### 部署管理

```java
/**
 * 查询部署列表
 */
public List<Deployment> listDeployments() {
    return repositoryService.createDeploymentQuery()
            .orderByDeploymenTime()
            .desc()
            .list();
}

/**
 * 根据名称查询部署
 */
public List<Deployment> listDeploymentsByName(String name) {
    return repositoryService.createDeploymentQuery()
            .deploymentName(name)
            .orderByDeploymenTime()
            .desc()
            .list();
}

/**
 * 获取部署资源
 */
public List<String> getDeploymentResources(String deploymentId) {
    return repositoryService.getDeploymentResourceNames(deploymentId);
}

/**
 * 获取部署资源内容
 */
public InputStream getDeploymentResource(String deploymentId, String resourceName) {
    return repositoryService.getResourceAsStream(deploymentId, resourceName);
}
```

### 模型管理

```java
import org.flowable.engine.repository.Model;

/**
 * 创建模型
 */
public Model createModel(String name, String key, String description) {
    Model model = repositoryService.newModel();
    model.setName(name);
    model.setKey(key);
    model.setCategory("process");
    repositoryService.saveModel(model);
    return model;
}

/**
 * 查询模型列表
 */
public List<Model> listModels() {
    return repositoryService.createModelQuery()
            .orderByModelName()
            .asc()
            .list();
}

/**
 * 根据Key查询模型
 */
public Model getModelByKey(String modelKey) {
    return repositoryService.createModelQuery()
            .modelKey(modelKey)
            .singleResult();
}

/**
 * 保存模型编辑器源
 */
public void saveModelEditorSource(String modelId, byte[] bytes) {
    repositoryService.addModelEditorSource(modelId, bytes);
}

/**
 * 获取模型编辑器源
 */
public byte[] getModelEditorSource(String modelId) {
    return repositoryService.getModelEditorSource(modelId);
}

/**
 * 删除模型
 */
public void deleteModel(String modelId) {
    repositoryService.deleteModel(modelId);
}
```

## 5.2 RuntimeService

RuntimeService 用于管理流程实例和执行。

### 流程实例管理

```java
import org.flowable.engine.RuntimeService;
import org.flowable.engine.runtime.ProcessInstance;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class RuntimeServiceExample {

    @Autowired
    private RuntimeService runtimeService;

    /**
     * 启动流程实例
     */
    public ProcessInstance startProcess(String processDefinitionKey) {
        return runtimeService.startProcessInstanceByKey(processDefinitionKey);
    }

    /**
     * 启动流程实例（带变量）
     */
    public ProcessInstance startProcess(String processDefinitionKey, Map<String, Object> variables) {
        return runtimeService.startProcessInstanceByKey(processDefinitionKey, variables);
    }

    /**
     * 启动流程实例（带业务Key）
     */
    public ProcessInstance startProcessWithBusinessKey(String processDefinitionKey, String businessKey) {
        return runtimeService.startProcessInstanceByKey(processDefinitionKey, businessKey);
    }

    /**
     * 启动流程实例（带业务Key和变量）
     */
    public ProcessInstance startProcess(String processDefinitionKey, String businessKey, Map<String, Object> variables) {
        return runtimeService.startProcessInstanceByKey(processDefinitionKey, businessKey, variables);
    }

    /**
     * 查询活跃的流程实例
     */
    public List<ProcessInstance> listActiveProcessInstances() {
        return runtimeService.createProcessInstanceQuery()
                .active()
                .orderByProcessInstanceId()
                .desc()
                .list();
    }

    /**
     * 根据流程定义Key查询流程实例
     */
    public List<ProcessInstance> listProcessInstancesByKey(String processDefinitionKey) {
        return runtimeService.createProcessInstanceQuery()
                .processDefinitionKey(processDefinitionKey)
                .orderByProcessInstanceId()
                .desc()
                .list();
    }

    /**
     * 根据业务Key查询流程实例
     */
    public ProcessInstance getProcessInstanceByBusinessKey(String processDefinitionKey, String businessKey) {
        return runtimeService.createProcessInstanceQuery()
                .processDefinitionKey(processDefinitionKey)
                .processInstanceBusinessKey(businessKey)
                .singleResult();
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
     * 获取流程实例的变量
     */
    public Map<String, Object> getVariables(String processInstanceId) {
        return runtimeService.getVariables(processInstanceId);
    }

    /**
     * 设置流程实例的变量
     */
    public void setVariables(String processInstanceId, Map<String, Object> variables) {
        runtimeService.setVariables(processInstanceId, variables);
    }

    /**
     * 获取单个变量
     */
    public Object getVariable(String processInstanceId, String variableName) {
        return runtimeService.getVariable(processInstanceId, variableName);
    }

    /**
     * 设置单个变量
     */
    public void setVariable(String processInstanceId, String variableName, Object value) {
        runtimeService.setVariable(processInstanceId, variableName, value);
    }
}
```

### 执行管理

```java
import org.flowable.engine.runtime.Execution;

/**
 * 查询流程实例的所有执行
 */
public List<Execution> listExecutionsByProcessInstanceId(String processInstanceId) {
    return runtimeService.createExecutionQuery()
            .processInstanceId(processInstanceId)
            .list();
}

/**
 * 查询活动的执行
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

/**
 * 信号事件触发
 */
public void signalEventReceived(String signalName) {
    runtimeService.signalEventReceived(signalName);
}

/**
 * 信号事件触发（指定执行）
 */
public void signalEventReceived(String signalName, String executionId) {
    runtimeService.signalEventReceived(signalName, executionId);
}

/**
 * 消息事件触发
 */
public void messageEventReceived(String messageName, String executionId) {
    runtimeService.messageEventReceived(messageName, executionId);
}
```

## 5.3 TaskService

TaskService 用于管理任务。

### 任务查询

```java
import org.flowable.engine.TaskService;
import org.flowable.task.api.Task;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TaskServiceExample {

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

    /**
     * 根据任务名称查询
     */
    public List<Task> listTasksByName(String taskName) {
        return taskService.createTaskQuery()
                .taskName(taskName)
                .orderByTaskCreateTime()
                .desc()
                .list();
    }

    /**
     * 根据任务Key查询
     */
    public List<Task> listTasksByTaskDefinitionKey(String taskDefinitionKey) {
        return taskService.createTaskQuery()
                .taskDefinitionKey(taskDefinitionKey)
                .orderByTaskCreateTime()
                .desc()
                .list();
    }

    /**
     * 获取单个任务
     */
    public Task getTask(String taskId) {
        return taskService.createTaskQuery()
                .taskId(taskId)
                .singleResult();
    }
}
```

### 任务办理

```java
import java.util.HashMap;
import java.util.Map;

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
 * 完成任务（带变量和本地变量）
 */
public void completeTask(String taskId, Map<String, Object> variables, Map<String, Object> localVariables) {
    taskService.complete(taskId, variables, localVariables);
}

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
public void setAssignee(String taskId, String userId) {
    taskService.setAssignee(taskId, userId);
}

/**
 * 解决任务
 */
public void resolveTask(String taskId) {
    taskService.resolveTask(taskId);
}

/**
 * 解决任务（带变量）
 */
public void resolveTask(String taskId, Map<String, Object> variables) {
    taskService.resolveTask(taskId, variables);
}
```

### 任务变量

```java
/**
 * 获取任务变量
 */
public Map<String, Object> getTaskVariables(String taskId) {
    return taskService.getVariables(taskId);
}

/**
 * 获取任务变量（指定变量名）
 */
public Object getTaskVariable(String taskId, String variableName) {
    return taskService.getVariable(taskId, variableName);
}

/**
 * 设置任务变量
 */
public void setTaskVariables(String taskId, Map<String, Object> variables) {
    taskService.setVariables(taskId, variables);
}

/**
 * 设置单个任务变量
 */
public void setTaskVariable(String taskId, String variableName, Object value) {
    taskService.setVariable(taskId, variableName, value);
}

/**
 * 获取任务本地变量
 */
public Map<String, Object> getTaskVariablesLocal(String taskId) {
    return taskService.getVariablesLocal(taskId);
}

/**
 * 设置任务本地变量
 */
public void setTaskVariablesLocal(String taskId, Map<String, Object> variables) {
    taskService.setVariablesLocal(taskId, variables);
}
```

### 任务附件和评论

```java
/**
 * 添加评论
 */
public void addComment(String taskId, String processInstanceId, String message) {
    taskService.addComment(taskId, processInstanceId, message);
}

/**
 * 获取任务评论
 */
public List<org.flowable.task.api.Comment> getTaskComments(String taskId) {
    return taskService.getTaskComments(taskId);
}

/**
 * 获取流程实例评论
 */
public List<org.flowable.task.api.Comment> getProcessInstanceComments(String processInstanceId) {
    return taskService.getProcessInstanceComments(processInstanceId);
}

/**
 * 创建附件
 */
public org.flowable.engine.common.api.attachment.Attachment createAttachment(
        String taskId, String processInstanceId, String attachmentType,
        String attachmentName, String attachmentDescription, InputStream content) {
    return taskService.createAttachment(attachmentType, taskId, processInstanceId,
            attachmentName, attachmentDescription, content);
}

/**
 * 创建附件（URL方式）
 */
public org.flowable.engine.common.api.attachment.Attachment createAttachmentUrl(
        String taskId, String processInstanceId, String attachmentType,
        String attachmentName, String attachmentDescription, String url) {
    return taskService.createAttachment(attachmentType, taskId, processInstanceId,
            attachmentName, attachmentDescription, url);
}

/**
 * 获取任务附件
 */
public List<org.flowable.engine.common.api.attachment.Attachment> getTaskAttachments(String taskId) {
    return taskService.getTaskAttachments(taskId);
}

/**
 * 获取流程实例附件
 */
public List<org.flowable.engine.common.api.attachment.Attachment> getProcessInstanceAttachments(String processInstanceId) {
    return taskService.getProcessInstanceAttachments(processInstanceId);
}

/**
 * 获取附件内容
 */
public InputStream getAttachmentContent(String attachmentId) {
    return taskService.getAttachmentContent(attachmentId);
}

/**
 * 删除附件
 */
public void deleteAttachment(String attachmentId) {
    taskService.deleteAttachment(attachmentId);
}
```

## 5.4 HistoryService

HistoryService 用于查询历史数据。

### 流程实例历史

```java
import org.flowable.engine.HistoryService;
import org.flowable.engine.history.HistoricProcessInstance;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class HistoryServiceExample {

    @Autowired
    private HistoryService historyService;

    /**
     * 查询所有历史流程实例
     */
    public List<HistoricProcessInstance> listHistoricProcessInstances() {
        return historyService.createHistoricProcessInstanceQuery()
                .orderByProcessInstanceStartTime()
                .desc()
                .list();
    }

    /**
     * 查询已完成的流程实例
     */
    public List<HistoricProcessInstance> listFinishedProcessInstances() {
        return historyService.createHistoricProcessInstanceQuery()
                .finished()
                .orderByProcessInstanceEndTime()
                .desc()
                .list();
    }

    /**
     * 查询未完成的流程实例
     */
    public List<HistoricProcessInstance> listUnfinishedProcessInstances() {
        return historyService.createHistoricProcessInstanceQuery()
                .unfinished()
                .orderByProcessInstanceStartTime()
                .desc()
                .list();
    }

    /**
     * 根据流程定义Key查询历史流程实例
     */
    public List<HistoricProcessInstance> listHistoricProcessInstancesByKey(String processDefinitionKey) {
        return historyService.createHistoricProcessInstanceQuery()
                .processDefinitionKey(processDefinitionKey)
                .orderByProcessInstanceStartTime()
                .desc()
                .list();
    }

    /**
     * 根据业务Key查询历史流程实例
     */
    public HistoricProcessInstance getHistoricProcessInstanceByBusinessKey(String processDefinitionKey, String businessKey) {
        return historyService.createHistoricProcessInstanceQuery()
                .processDefinitionKey(processDefinitionKey)
                .processInstanceBusinessKey(businessKey)
                .singleResult();
    }

    /**
     * 根据发起人查询历史流程实例
     */
    public List<HistoricProcessInstance> listHistoricProcessInstancesByStartUser(String startUserId) {
        return historyService.createHistoricProcessInstanceQuery()
                .startedBy(startUserId)
                .orderByProcessInstanceStartTime()
                .desc()
                .list();
    }

    /**
     * 获取单个历史流程实例
     */
    public HistoricProcessInstance getHistoricProcessInstance(String processInstanceId) {
        return historyService.createHistoricProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .singleResult();
    }

    /**
     * 删除历史流程实例
     */
    public void deleteHistoricProcessInstance(String processInstanceId) {
        historyService.deleteHistoricProcessInstance(processInstanceId);
    }
}
```

### 活动实例历史

```java
import org.flowable.engine.history.HistoricActivityInstance;

/**
 * 查询流程实例的历史活动
 */
public List<HistoricActivityInstance> listHistoricActivityInstances(String processInstanceId) {
    return historyService.createHistoricActivityInstanceQuery()
            .processInstanceId(processInstanceId)
            .orderByHistoricActivityInstanceStartTime()
            .asc()
            .list();
}

/**
 * 查询已完成的历史活动
 */
public List<HistoricActivityInstance> listFinishedHistoricActivityInstances(String processInstanceId) {
    return historyService.createHistoricActivityInstanceQuery()
            .processInstanceId(processInstanceId)
            .finished()
            .orderByHistoricActivityInstanceEndTime()
            .desc()
            .list();
}

/**
 * 根据活动类型查询
 */
public List<HistoricActivityInstance> listHistoricActivityInstancesByType(String processInstanceId, String activityType) {
    return historyService.createHistoricActivityInstanceQuery()
            .processInstanceId(processInstanceId)
            .activityType(activityType)
            .orderByHistoricActivityInstanceStartTime()
            .asc()
            .list();
}

/**
 * 根据办理人查询
 */
public List<HistoricActivityInstance> listHistoricActivityInstancesByAssignee(String assignee) {
    return historyService.createHistoricActivityInstanceQuery()
            .taskAssignee(assignee)
            .orderByHistoricActivityInstanceStartTime()
            .desc()
            .list();
}
```

### 任务历史

```java
import org.flowable.engine.history.HistoricTaskInstance;

/**
 * 查询历史任务
 */
public List<HistoricTaskInstance> listHistoricTaskInstances() {
    return historyService.createHistoricTaskInstanceQuery()
            .orderByHistoricTaskInstanceEndTime()
            .desc()
            .list();
}

/**
 * 查询已完成的历史任务
 */
public List<HistoricTaskInstance> listFinishedHistoricTaskInstances() {
    return historyService.createHistoricTaskInstanceQuery()
            .finished()
            .orderByHistoricTaskInstanceEndTime()
            .desc()
            .list();
}

/**
 * 查询办理人的历史任务
 */
public List<HistoricTaskInstance> listHistoricTaskInstancesByAssignee(String assignee) {
    return historyService.createHistoricTaskInstanceQuery()
            .taskAssignee(assignee)
            .orderByHistoricTaskInstanceEndTime()
            .desc()
            .list();
}

/**
 * 查询流程实例的历史任务
 */
public List<HistoricTaskInstance> listHistoricTaskInstancesByProcessInstanceId(String processInstanceId) {
    return historyService.createHistoricTaskInstanceQuery()
            .processInstanceId(processInstanceId)
            .orderByHistoricTaskInstanceEndTime()
            .desc()
            .list();
}

/**
 * 根据任务名称查询历史任务
 */
public List<HistoricTaskInstance> listHistoricTaskInstancesByName(String taskName) {
    return historyService.createHistoricTaskInstanceQuery()
            .taskName(taskName)
            .orderByHistoricTaskInstanceEndTime()
            .desc()
            .list();
}

/**
 * 获取单个历史任务
 */
public HistoricTaskInstance getHistoricTaskInstance(String taskId) {
    return historyService.createHistoricTaskInstanceQuery()
            .taskId(taskId)
            .singleResult();
}
```

### 变量历史

```java
import org.flowable.engine.history.HistoricVariableInstance;

/**
 * 查询流程实例的历史变量
 */
public List<HistoricVariableInstance> listHistoricVariableInstances(String processInstanceId) {
    return historyService.createHistoricVariableInstanceQuery()
            .processInstanceId(processInstanceId)
            .orderByVariableName()
            .asc()
            .list();
}

/**
 * 查询任务的历史变量
 */
public List<HistoricVariableInstance> listHistoricVariableInstancesByTaskId(String taskId) {
    return historyService.createHistoricVariableInstanceQuery()
            .taskId(taskId)
            .orderByVariableName()
            .asc()
            .list();
}

/**
 * 根据变量名查询
 */
public List<HistoricVariableInstance> listHistoricVariableInstancesByName(String variableName) {
    return historyService.createHistoricVariableInstanceQuery()
            .variableName(variableName)
            .orderByProcessInstanceId()
            .asc()
            .list();
}
```

### 详情历史

```java
import org.flowable.engine.history.HistoricDetail;

/**
 * 查询流程实例的历史详情
 */
public List<HistoricDetail> listHistoricDetails(String processInstanceId) {
    return historyService.createHistoricDetailQuery()
            .processInstanceId(processInstanceId)
            .orderByTime()
            .desc()
            .list();
}

/**
 * 查询变量更新历史
 */
public List<HistoricDetail> listHistoricVariableUpdates(String processInstanceId) {
    return historyService.createHistoricDetailQuery()
            .processInstanceId(processInstanceId)
            .variableUpdates()
            .orderByTime()
            .desc()
            .list();
}

/**
 * 查询表单属性历史
 */
public List<HistoricDetail> listHistoricFormProperties(String processInstanceId) {
    return historyService.createHistoricDetailQuery()
            .processInstanceId(processInstanceId)
            .formProperties()
            .orderByTime()
            .desc()
            .list();
}
```

## 5.5 IdentityService

IdentityService 用于管理用户和用户组。

### 用户管理

```java
import org.flowable.engine.IdentityService;
import org.flowable.idm.api.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class IdentityServiceExample {

    @Autowired
    private IdentityService identityService;

    /**
     * 创建用户
     */
    public User createUser(String userId, String firstName, String lastName, String email, String password) {
        User user = identityService.newUser(userId);
        user.setFirstName(firstName);
        user.setLastName(lastName);
        user.setEmail(email);
        user.setPassword(password);
        identityService.saveUser(user);
        return user;
    }

    /**
     * 查询所有用户
     */
    public List<User> listUsers() {
        return identityService.createUserQuery()
                .orderByUserId()
                .asc()
                .list();
    }

    /**
     * 根据ID查询用户
     */
    public User getUser(String userId) {
        return identityService.createUserQuery()
                .userId(userId)
                .singleResult();
    }

    /**
     * 根据姓名查询用户
     */
    public List<User> listUsersByName(String firstName, String lastName) {
        return identityService.createUserQuery()
                .userFirstName(firstName)
                .userLastName(lastName)
                .orderByUserId()
                .asc()
                .list();
    }

    /**
     * 更新用户
     */
    public void updateUser(User user) {
        identityService.saveUser(user);
    }

    /**
     * 删除用户
     */
    public void deleteUser(String userId) {
        identityService.deleteUser(userId);
    }

    /**
     * 验证用户密码
     */
    public boolean checkPassword(String userId, String password) {
        return identityService.checkPassword(userId, password);
    }

    /**
     * 设置当前认证用户
     */
    public void setAuthenticatedUserId(String userId) {
        identityService.setAuthenticatedUserId(userId);
    }
}
```

### 用户组管理

```java
import org.flowable.idm.api.Group;

/**
 * 创建用户组
 */
public Group createGroup(String groupId, String name, String type) {
    Group group = identityService.newGroup(groupId);
    group.setName(name);
    group.setType(type);
    identityService.saveGroup(group);
    return group;
}

/**
 * 查询所有用户组
 */
public List<Group> listGroups() {
    return identityService.createGroupQuery()
            .orderByGroupId()
            .asc()
            .list();
}

/**
 * 根据ID查询用户组
 */
public Group getGroup(String groupId) {
    return identityService.createGroupQuery()
            .groupId(groupId)
            .singleResult();
}

/**
 * 根据类型查询用户组
 */
public List<Group> listGroupsByType(String type) {
    return identityService.createGroupQuery()
            .groupType(type)
            .orderByGroupId()
            .asc()
            .list();
}

/**
 * 根据名称查询用户组
 */
public List<Group> listGroupsByName(String name) {
    return identityService.createGroupQuery()
            .groupName(name)
            .orderByGroupId()
            .asc()
            .list();
}

/**
 * 更新用户组
 */
public void updateGroup(Group group) {
    identityService.saveGroup(group);
}

/**
 * 删除用户组
 */
public void deleteGroup(String groupId) {
    identityService.deleteGroup(groupId);
}
```

### 权限管理

```java
/**
 * 添加用户到用户组
 */
public void addUserToGroup(String userId, String groupId) {
    identityService.createMembership(userId, groupId);
}

/**
 * 从用户组移除用户
 */
public void removeUserFromGroup(String userId, String groupId) {
    identityService.deleteMembership(userId, groupId);
}

/**
 * 查询用户所属的用户组
 */
public List<Group> listGroupsByUser(String userId) {
    return identityService.createGroupQuery()
            .groupMember(userId)
            .orderByGroupId()
            .asc()
            .list();
}

/**
 * 查询用户组的用户
 */
public List<User> listUsersByGroup(String groupId) {
    return identityService.createUserQuery()
            .memberOfGroup(groupId)
            .orderByUserId()
            .asc()
            .list();
}

/**
 * 检查用户是否属于某个用户组
 */
public boolean isUserInGroup(String userId, String groupId) {
    Group group = identityService.createGroupQuery()
            .groupId(groupId)
            .groupMember(userId)
            .singleResult();
    return group != null;
}
```
