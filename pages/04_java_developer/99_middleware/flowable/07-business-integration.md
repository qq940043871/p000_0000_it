# 业务系统深度集成

## 7.1 架构设计

### 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                        表现层 (Presentation)                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Web管理后台  │  │  移动端APP   │  │  第三方集成   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        应用层 (Application)                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              工作流服务 (Workflow Service)            │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  │
│  │  │流程管理  │  │任务管理  │  │表单管理  │        │  │
│  │  └──────────┘  └──────────┘  └──────────┘        │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              业务服务 (Business Service)              │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  │
│  │  │用户服务  │  │组织服务  │  │权限服务  │        │  │
│  │  └──────────┘  └──────────┘  └──────────┘        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        领域层 (Domain)                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              流程引擎 (Flowable Engine)               │  │
│  │  RepositoryService, RuntimeService, TaskService...  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              业务领域 (Business Domain)                │  │
│  │  用户实体, 组织实体, 业务实体, 表单实体...            │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                        基础设施层 (Infrastructure)           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ MySQL    │  │ Redis    │  │ RabbitMQ │  │ MinIO    │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 服务封装

#### 工作流服务接口

```java
public interface WorkflowService {

    /**
     * 启动流程
     */
    ProcessInstanceDTO startProcess(String processDefinitionKey, String businessKey, 
                                     Map<String, Object> variables);

    /**
     * 完成任务
     */
    void completeTask(String taskId, Map<String, Object> variables);

    /**
     * 查询待办任务
     */
    PageResult<TaskDTO> listTodoTasks(String userId, PageQuery pageQuery);

    /**
     * 查询已办任务
     */
    PageResult<TaskDTO> listDoneTasks(String userId, PageQuery pageQuery);

    /**
     * 查询流程实例
     */
    ProcessInstanceDTO getProcessInstance(String processInstanceId);

    /**
     * 撤回流程
     */
    void withdrawProcess(String processInstanceId);

    /**
     * 委托任务
     */
    void delegateTask(String taskId, String userId);

    /**
     * 转办任务
     */
    void transferTask(String taskId, String userId);

    /**
     * 添加评论
     */
    void addComment(String taskId, String comment);

    /**
     * 获取流程图
     */
    byte[] getProcessDiagram(String processInstanceId);
}
```

#### 工作流服务实现

```java
@Service
public class WorkflowServiceImpl implements WorkflowService {

    @Autowired
    private RepositoryService repositoryService;

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private IdentityService identityService;

    @Override
    public ProcessInstanceDTO startProcess(String processDefinitionKey, String businessKey, 
                                             Map<String, Object> variables) {
        // 设置当前用户
        String currentUserId = SecurityUtils.getCurrentUserId();
        identityService.setAuthenticatedUserId(currentUserId);

        // 添加发起人变量
        variables.put("startUserId", currentUserId);
        variables.put("startTime", new Date());

        // 启动流程
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                processDefinitionKey, businessKey, variables);

        // 保存业务关联
        saveBusinessRelation(processInstance.getId(), businessKey, processDefinitionKey);

        // 发送流程启动事件
        publishProcessStartEvent(processInstance);

        return convertToDTO(processInstance);
    }

    @Override
    public void completeTask(String taskId, Map<String, Object> variables) {
        // 获取任务
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        if (task == null) {
            throw new BusinessException("任务不存在");
        }

        // 检查权限
        String currentUserId = SecurityUtils.getCurrentUserId();
        if (!canCompleteTask(currentUserId, task)) {
            throw new BusinessException("没有任务办理权限");
        }

        // 添加办理人变量
        variables.put("assignee", currentUserId);
        variables.put("completeTime", new Date());

        // 完成任务
        taskService.complete(taskId, variables);

        // 发送任务完成事件
        publishTaskCompleteEvent(task);
    }

    @Override
    public PageResult<TaskDTO> listTodoTasks(String userId, PageQuery pageQuery) {
        // 查询待办任务
        List<Task> tasks = taskService.createTaskQuery()
                .taskAssignee(userId)
                .active()
                .orderByTaskCreateTime()
                .desc()
                .listPage(pageQuery.getOffset(), pageQuery.getPageSize());

        // 查询总数
        long total = taskService.createTaskQuery()
                .taskAssignee(userId)
                .active()
                .count();

        // 转换为DTO
        List<TaskDTO> taskDTOs = tasks.stream()
                .map(this::convertToTaskDTO)
                .collect(Collectors.toList());

        return PageResult.of(taskDTOs, total, pageQuery);
    }

    private boolean canCompleteTask(String userId, Task task) {
        // 检查是否是办理人
        if (userId.equals(task.getAssignee())) {
            return true;
        }

        // 检查是否是候选人
        List<IdentityLink> identityLinks = taskService.getIdentityLinksForTask(task.getId());
        for (IdentityLink identityLink : identityLinks) {
            if (userId.equals(identityLink.getUserId())) {
                return true;
            }
            if (identityLink.getGroupId() != null) {
                if (isUserInGroup(userId, identityLink.getGroupId())) {
                    return true;
                }
            }
        }

        return false;
    }

    private boolean isUserInGroup(String userId, String groupId) {
        return identityService.createGroupQuery()
                .groupId(groupId)
                .groupMember(userId)
                .singleResult() != null;
    }
}
```

### 接口设计

#### REST API 接口

```java
@RestController
@RequestMapping("/api/workflow")
public class WorkflowController {

    @Autowired
    private WorkflowService workflowService;

    /**
     * 启动流程
     */
    @PostMapping("/process/start")
    public Result<ProcessInstanceDTO> startProcess(@RequestBody StartProcessRequest request) {
        ProcessInstanceDTO processInstance = workflowService.startProcess(
                request.getProcessDefinitionKey(),
                request.getBusinessKey(),
                request.getVariables()
        );
        return Result.success(processInstance);
    }

    /**
     * 完成任务
     */
    @PostMapping("/task/{taskId}/complete")
    public Result<Void> completeTask(@PathVariable String taskId, 
                                       @RequestBody CompleteTaskRequest request) {
        workflowService.completeTask(taskId, request.getVariables());
        return Result.success();
    }

    /**
     * 查询待办任务
     */
    @GetMapping("/task/todo")
    public Result<PageResult<TaskDTO>> listTodoTasks(PageQuery pageQuery) {
        String userId = SecurityUtils.getCurrentUserId();
        PageResult<TaskDTO> tasks = workflowService.listTodoTasks(userId, pageQuery);
        return Result.success(tasks);
    }

    /**
     * 查询已办任务
     */
    @GetMapping("/task/done")
    public Result<PageResult<TaskDTO>> listDoneTasks(PageQuery pageQuery) {
        String userId = SecurityUtils.getCurrentUserId();
        PageResult<TaskDTO> tasks = workflowService.listDoneTasks(userId, pageQuery);
        return Result.success(tasks);
    }

    /**
     * 获取流程实例详情
     */
    @GetMapping("/process/{processInstanceId}")
    public Result<ProcessInstanceDTO> getProcessInstance(@PathVariable String processInstanceId) {
        ProcessInstanceDTO processInstance = workflowService.getProcessInstance(processInstanceId);
        return Result.success(processInstance);
    }

    /**
     * 撤回流程
     */
    @PostMapping("/process/{processInstanceId}/withdraw")
    public Result<Void> withdrawProcess(@PathVariable String processInstanceId) {
        workflowService.withdrawProcess(processInstanceId);
        return Result.success();
    }

    /**
     * 委托任务
     */
    @PostMapping("/task/{taskId}/delegate")
    public Result<Void> delegateTask(@PathVariable String taskId, 
                                       @RequestParam String userId) {
        workflowService.delegateTask(taskId, userId);
        return Result.success();
    }

    /**
     * 转办任务
     */
    @PostMapping("/task/{taskId}/transfer")
    public Result<Void> transferTask(@PathVariable String taskId, 
                                       @RequestParam String userId) {
        workflowService.transferTask(taskId, userId);
        return Result.success();
    }

    /**
     * 添加评论
     */
    @PostMapping("/task/{taskId}/comment")
    public Result<Void> addComment(@PathVariable String taskId, 
                                    @RequestBody AddCommentRequest request) {
        workflowService.addComment(taskId, request.getComment());
        return Result.success();
    }

    /**
     * 获取流程图
     */
    @GetMapping(value = "/process/{processInstanceId}/diagram", produces = MediaType.IMAGE_PNG_VALUE)
    public byte[] getProcessDiagram(@PathVariable String processInstanceId) {
        return workflowService.getProcessDiagram(processInstanceId);
    }
}
```

## 7.2 数据集成

### 业务数据关联

#### 业务关联表设计

```sql
-- 流程业务关联表
CREATE TABLE wf_business_relation (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
    process_instance_id VARCHAR(64) NOT NULL COMMENT '流程实例ID',
    business_key VARCHAR(128) NOT NULL COMMENT '业务Key',
    business_type VARCHAR(64) NOT NULL COMMENT '业务类型',
    process_definition_key VARCHAR(128) NOT NULL COMMENT '流程定义Key',
    start_user_id VARCHAR(64) NOT NULL COMMENT '发起人ID',
    start_time DATETIME NOT NULL COMMENT '发起时间',
    end_time DATETIME COMMENT '结束时间',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态:1-进行中,2-已完成,3-已终止',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    UNIQUE KEY uk_process_instance (process_instance_id),
    KEY idx_business_key (business_key, business_type),
    KEY idx_start_user (start_user_id),
    KEY idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='流程业务关联表';

-- 流程操作记录表
CREATE TABLE wf_operation_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
    process_instance_id VARCHAR(64) NOT NULL COMMENT '流程实例ID',
    task_id VARCHAR(64) COMMENT '任务ID',
    operator_id VARCHAR(64) NOT NULL COMMENT '操作人ID',
    operator_name VARCHAR(64) NOT NULL COMMENT '操作人姓名',
    operation_type VARCHAR(32) NOT NULL COMMENT '操作类型',
    operation_comment TEXT COMMENT '操作意见',
    operation_time DATETIME NOT NULL COMMENT '操作时间',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    KEY idx_process_instance (process_instance_id),
    KEY idx_task (task_id),
    KEY idx_operator (operator_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='流程操作记录表';
```

#### 业务关联服务

```java
@Service
public class BusinessRelationService {

    @Autowired
    private BusinessRelationMapper businessRelationMapper;

    @Autowired
    private OperationLogMapper operationLogMapper;

    /**
     * 保存业务关联
     */
    @Transactional(rollbackFor = Exception.class)
    public void saveBusinessRelation(String processInstanceId, String businessKey, 
                                       String processDefinitionKey) {
        BusinessRelation relation = new BusinessRelation();
        relation.setProcessInstanceId(processInstanceId);
        relation.setBusinessKey(businessKey);
        relation.setBusinessType(extractBusinessType(processDefinitionKey));
        relation.setProcessDefinitionKey(processDefinitionKey);
        relation.setStartUserId(SecurityUtils.getCurrentUserId());
        relation.setStartTime(new Date());
        relation.setStatus(1);
        businessRelationMapper.insert(relation);
    }

    /**
     * 更新流程状态
     */
    @Transactional(rollbackFor = Exception.class)
    public void updateProcessStatus(String processInstanceId, Integer status) {
        BusinessRelation relation = businessRelationMapper.selectByProcessInstanceId(processInstanceId);
        if (relation != null) {
            relation.setStatus(status);
            if (status == 2 || status == 3) {
                relation.setEndTime(new Date());
            }
            businessRelationMapper.updateById(relation);
        }
    }

    /**
     * 根据业务Key查询流程实例
     */
    public BusinessRelation getByBusinessKey(String businessKey, String businessType) {
        return businessRelationMapper.selectByBusinessKey(businessKey, businessType);
    }

    /**
     * 记录操作日志
     */
    @Transactional(rollbackFor = Exception.class)
    public void recordOperation(String processInstanceId, String taskId, 
                                  String operationType, String comment) {
        OperationLog log = new OperationLog();
        log.setProcessInstanceId(processInstanceId);
        log.setTaskId(taskId);
        log.setOperatorId(SecurityUtils.getCurrentUserId());
        log.setOperatorName(SecurityUtils.getCurrentUserName());
        log.setOperationType(operationType);
        log.setOperationComment(comment);
        log.setOperationTime(new Date());
        operationLogMapper.insert(log);
    }

    /**
     * 查询流程操作日志
     */
    public List<OperationLog> getOperationLogs(String processInstanceId) {
        return operationLogMapper.selectByProcessInstanceId(processInstanceId);
    }

    private String extractBusinessType(String processDefinitionKey) {
        // 根据流程定义Key提取业务类型
        return processDefinitionKey.split("_")[0];
    }
}
```

### 表单集成

#### 表单设计器集成

```vue
<template>
  <div class="form-designer">
    <div class="designer-header">
      <el-button type="primary" @click="saveForm">保存表单</el-button>
      <el-button @click="previewForm">预览表单</el-button>
    </div>
    
    <div class="designer-content">
      <div class="component-palette">
        <div 
          v-for="component in componentList" 
          :key="component.type"
          class="palette-item"
          draggable="true"
          @dragstart="onDragStart($event, component)"
        >
          <el-icon :size="20"><component :is="component.icon" /></el-icon>
          <span>{{ component.label }}</span>
        </div>
      </div>
      
      <div class="form-canvas" @drop="onDrop" @dragover.prevent>
        <div 
          v-for="(field, index) in formFields" 
          :key="field.id"
          class="form-field"
          @click="selectField(index)"
          :class="{ selected: selectedIndex === index }"
        >
          <component :is="field.component" v-bind="field.props" />
        </div>
      </div>
      
      <div class="property-panel">
        <div v-if="selectedField">
          <el-form label-position="top">
            <el-form-item label="字段名称">
              <el-input v-model="selectedField.props.label" />
            </el-form-item>
            <el-form-item label="字段标识">
              <el-input v-model="selectedField.props.fieldKey" />
            </el-form-item>
            <el-form-item label="是否必填">
              <el-switch v-model="selectedField.props.required" />
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { 
  Document, 
  Edit, 
  User, 
  Tickets, 
  Calendar, 
  Picture 
} from '@element-plus/icons-vue'

const componentList = ref([
  { type: 'input', label: '单行输入', icon: Edit },
  { type: 'textarea', label: '多行输入', icon: Document },
  { type: 'select', label: '下拉选择', icon: Tickets },
  { type: 'date', label: '日期选择', icon: Calendar },
  { type: 'user', label: '人员选择', icon: User },
  { type: 'file', label: '文件上传', icon: Picture }
])

const formFields = ref([])
const selectedIndex = ref(-1)
const selectedField = ref(null)

const onDragStart = (event, component) => {
  event.dataTransfer.setData('component', JSON.stringify(component))
}

const onDrop = (event) => {
  const componentData = JSON.parse(event.dataTransfer.getData('component'))
  const field = {
    id: Date.now().toString(),
    component: componentData.type,
    props: {
      label: componentData.label,
      fieldKey: componentData.type + '_' + Date.now(),
      required: false
    }
  }
  formFields.value.push(field)
}

const selectField = (index) => {
  selectedIndex.value = index
  selectedField.value = formFields.value[index]
}

const saveForm = () => {
  console.log('保存表单:', formFields.value)
}

const previewForm = () => {
  console.log('预览表单:', formFields.value)
}
</script>

<style scoped>
.form-designer {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.designer-header {
  padding: 16px;
  background: white;
  border-bottom: 1px solid #e4e7ed;
  display: flex;
  gap: 12px;
}

.designer-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.component-palette {
  width: 200px;
  background: white;
  border-right: 1px solid #e4e7ed;
  padding: 16px;
  overflow-y: auto;
}

.palette-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  margin-bottom: 8px;
  border-radius: 4px;
  cursor: grab;
  transition: all 0.2s;
}

.palette-item:hover {
  background: #ecf5ff;
  color: #409eff;
}

.form-canvas {
  flex: 1;
  padding: 24px;
  background: #f5f7fa;
  overflow-y: auto;
}

.form-field {
  background: white;
  padding: 16px;
  margin-bottom: 16px;
  border-radius: 4px;
  border: 2px solid transparent;
  cursor: pointer;
  transition: all 0.2s;
}

.form-field.selected {
  border-color: #409eff;
}

.property-panel {
  width: 280px;
  background: white;
  border-left: 1px solid #e4e7ed;
  padding: 16px;
  overflow-y: auto;
}
</style>
```

#### 表单数据绑定

```java
@Service
public class FormDataService {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Autowired
    private FormRepository formRepository;

    /**
     * 获取启动表单数据
     */
    public FormData getStartFormData(String processDefinitionId) {
        FormDefinition formDefinition = formRepository.getByProcessDefinitionId(processDefinitionId);
        if (formDefinition == null) {
            return new FormData();
        }

        // 初始化表单数据
        FormData formData = new FormData();
        formData.setFormDefinition(formDefinition);
        formData.setFormValues(initFormValues(formDefinition));

        return formData;
    }

    /**
     * 获取任务表单数据
     */
    public FormData getTaskFormData(String taskId) {
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        if (task == null) {
            throw new BusinessException("任务不存在");
        }

        // 获取流程变量
        Map<String, Object> variables = taskService.getVariables(taskId);

        // 获取表单定义
        FormDefinition formDefinition = formRepository.getByTaskDefinitionKey(task.getTaskDefinitionKey());
        if (formDefinition == null) {
            return new FormData();
        }

        // 绑定表单数据
        FormData formData = new FormData();
        formData.setFormDefinition(formDefinition);
        formData.setFormValues(bindFormValues(formDefinition, variables));

        return formData;
    }

    /**
     * 保存表单数据并启动流程
     */
    public ProcessInstanceDTO submitStartForm(String processDefinitionId, 
                                                Map<String, Object> formValues) {
        // 验证表单数据
        FormDefinition formDefinition = formRepository.getByProcessDefinitionId(processDefinitionId);
        validateFormData(formDefinition, formValues);

        // 启动流程
        ProcessInstance processInstance = runtimeService.startProcessInstanceWithForm(
                processDefinitionId, null, formValues, null);

        return convertToDTO(processInstance);
    }

    /**
     * 保存表单数据并完成任务
     */
    public void submitTaskForm(String taskId, Map<String, Object> formValues) {
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        if (task == null) {
            throw new BusinessException("任务不存在");
        }

        // 验证表单数据
        FormDefinition formDefinition = formRepository.getByTaskDefinitionKey(task.getTaskDefinitionKey());
        validateFormData(formDefinition, formValues);

        // 完成任务
        taskService.complete(taskId, formValues);
    }

    private Map<String, Object> initFormValues(FormDefinition formDefinition) {
        Map<String, Object> values = new HashMap<>();
        for (FormField field : formDefinition.getFields()) {
            if (field.getDefaultValue() != null) {
                values.put(field.getFieldKey(), field.getDefaultValue());
            }
        }
        return values;
    }

    private Map<String, Object> bindFormValues(FormDefinition formDefinition, 
                                                  Map<String, Object> variables) {
        Map<String, Object> values = new HashMap<>();
        for (FormField field : formDefinition.getFields()) {
            Object value = variables.get(field.getFieldKey());
            if (value != null) {
                values.put(field.getFieldKey(), value);
            }
        }
        return values;
    }

    private void validateFormData(FormDefinition formDefinition, 
                                    Map<String, Object> formValues) {
        for (FormField field : formDefinition.getFields()) {
            if (field.isRequired()) {
                Object value = formValues.get(field.getFieldKey());
                if (value == null || (value instanceof String && ((String) value).isEmpty())) {
                    throw new BusinessException(field.getLabel() + "不能为空");
                }
            }
        }
    }
}
```

## 7.3 事件驱动

### 流程事件监听

#### 全局事件监听器

```java
@Component
public class GlobalEventListener implements FlowableEventListener {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Override
    public void onEvent(FlowableEvent event) {
        switch (event.getType()) {
            case PROCESS_STARTED:
                handleProcessStarted((FlowableEntityEvent) event);
                break;
            case PROCESS_COMPLETED:
                handleProcessCompleted((FlowableEntityEvent) event);
                break;
            case TASK_CREATED:
                handleTaskCreated((FlowableEntityEvent) event);
                break;
            case TASK_ASSIGNED:
                handleTaskAssigned((FlowableEntityEvent) event);
                break;
            case TASK_COMPLETED:
                handleTaskCompleted((FlowableEntityEvent) event);
                break;
            case ACTIVITY_STARTED:
                handleActivityStarted((FlowableEntityEvent) event);
                break;
            case ACTIVITY_COMPLETED:
                handleActivityCompleted((FlowableEntityEvent) event);
                break;
            default:
                break;
        }
    }

    private void handleProcessStarted(FlowableEntityEvent event) {
        ProcessInstance processInstance = (ProcessInstance) event.getEntity();
        ProcessStartedEvent springEvent = new ProcessStartedEvent(this, processInstance);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleProcessCompleted(FlowableEntityEvent event) {
        HistoricProcessInstance historicProcessInstance = 
            (HistoricProcessInstance) event.getEntity();
        ProcessCompletedEvent springEvent = 
            new ProcessCompletedEvent(this, historicProcessInstance);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleTaskCreated(FlowableEntityEvent event) {
        Task task = (Task) event.getEntity();
        TaskCreatedEvent springEvent = new TaskCreatedEvent(this, task);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleTaskAssigned(FlowableEntityEvent event) {
        Task task = (Task) event.getEntity();
        TaskAssignedEvent springEvent = new TaskAssignedEvent(this, task);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleTaskCompleted(FlowableEntityEvent event) {
        HistoricTaskInstance historicTaskInstance = 
            (HistoricTaskInstance) event.getEntity();
        TaskCompletedEvent springEvent = 
            new TaskCompletedEvent(this, historicTaskInstance);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleActivityStarted(FlowableEntityEvent event) {
        ActivityInstance activityInstance = (ActivityInstance) event.getEntity();
        ActivityStartedEvent springEvent = new ActivityStartedEvent(this, activityInstance);
        eventPublisher.publishEvent(springEvent);
    }

    private void handleActivityCompleted(FlowableEntityEvent event) {
        ActivityInstance activityInstance = (ActivityInstance) event.getEntity();
        ActivityCompletedEvent springEvent = new ActivityCompletedEvent(this, activityInstance);
        eventPublisher.publishEvent(springEvent);
    }

    @Override
    public boolean isFailOnException() {
        return false;
    }

    @Override
    public boolean isFireOnTransactionLifecycleEvent() {
        return false;
    }

    @Override
    public String getOnTransaction() {
        return null;
    }
}
```

#### 注册事件监听器

```java
@Configuration
public class FlowableConfig {

    @Autowired
    private GlobalEventListener globalEventListener;

    @Bean
    public EngineConfigurationConfigurer<SpringProcessEngineConfiguration> 
            engineConfigurationConfigurer() {
        return engineConfiguration -> {
            // 注册全局事件监听器
            List<FlowableEventListener> listeners = new ArrayList<>();
            listeners.add(globalEventListener);
            engineConfiguration.setEventListeners(listeners);
        };
    }
}
```

### 业务事件触发

#### Spring 事件定义

```java
public class ProcessStartedEvent extends ApplicationEvent {
    private final ProcessInstance processInstance;

    public ProcessStartedEvent(Object source, ProcessInstance processInstance) {
        super(source);
        this.processInstance = processInstance;
    }

    public ProcessInstance getProcessInstance() {
        return processInstance;
    }
}

public class ProcessCompletedEvent extends ApplicationEvent {
    private final HistoricProcessInstance historicProcessInstance;

    public ProcessCompletedEvent(Object source, HistoricProcessInstance historicProcessInstance) {
        super(source);
        this.historicProcessInstance = historicProcessInstance;
    }

    public HistoricProcessInstance getHistoricProcessInstance() {
        return historicProcessInstance;
    }
}

public class TaskCreatedEvent extends ApplicationEvent {
    private final Task task;

    public TaskCreatedEvent(Object source, Task task) {
        super(source);
        this.task = task;
    }

    public Task getTask() {
        return task;
    }
}

public class TaskAssignedEvent extends ApplicationEvent {
    private final Task task;

    public TaskAssignedEvent(Object source, Task task) {
        super(source);
        this.task = task;
    }

    public Task getTask() {
        return task;
    }
}

public class TaskCompletedEvent extends ApplicationEvent {
    private final HistoricTaskInstance historicTaskInstance;

    public TaskCompletedEvent(Object source, HistoricTaskInstance historicTaskInstance) {
        super(source);
        this.historicTaskInstance = historicTaskInstance;
    }

    public HistoricTaskInstance getHistoricTaskInstance() {
        return historicTaskInstance;
    }
}
```

#### 业务事件监听器

```java
@Component
public class BusinessEventListener {

    @Autowired
    private NotificationService notificationService;

    @Autowired
    private BusinessStatusService businessStatusService;

    @Autowired
    private BusinessRelationService businessRelationService;

    /**
     * 处理流程启动事件
     */
    @EventListener
    @Async
    public void handleProcessStarted(ProcessStartedEvent event) {
        ProcessInstance processInstance = event.getProcessInstance();
        
        // 更新业务状态
        businessStatusService.updateStatus(
                processInstance.getBusinessKey(),
                "PROCESSING"
        );
        
        // 记录操作日志
        businessRelationService.recordOperation(
                processInstance.getId(),
                null,
                "PROCESS_START",
                "流程启动"
        );
    }

    /**
     * 处理流程完成事件
     */
    @EventListener
    @Async
    public void handleProcessCompleted(ProcessCompletedEvent event) {
        HistoricProcessInstance historicProcessInstance = event.getHistoricProcessInstance();
        
        // 更新业务状态
        businessStatusService.updateStatus(
                historicProcessInstance.getBusinessKey(),
                "COMPLETED"
        );
        
        // 更新流程关联状态
        businessRelationService.updateProcessStatus(
                historicProcessInstance.getId(),
                2
        );
        
        // 记录操作日志
        businessRelationService.recordOperation(
                historicProcessInstance.getId(),
                null,
                "PROCESS_COMPLETE",
                "流程完成"
        );
        
        // 发送完成通知
        notificationService.sendProcessCompleteNotification(
                historicProcessInstance.getStartUserId(),
                historicProcessInstance.getId()
        );
    }

    /**
     * 处理任务创建事件
     */
    @EventListener
    @Async
    public void handleTaskCreated(TaskCreatedEvent event) {
        Task task = event.getTask();
        
        // 如果任务已分配，发送通知
        if (task.getAssignee() != null) {
            notificationService.sendTaskNotification(
                    task.getAssignee(),
                    task.getId(),
                    task.getName()
            );
        }
    }

    /**
     * 处理任务分配事件
     */
    @EventListener
    @Async
    public void handleTaskAssigned(TaskAssignedEvent event) {
        Task task = event.getTask();
        
        // 发送任务分配通知
        notificationService.sendTaskAssignedNotification(
                task.getAssignee(),
                task.getId(),
                task.getName()
        );
    }

    /**
     * 处理任务完成事件
     */
    @EventListener
    @Async
    public void handleTaskCompleted(TaskCompletedEvent event) {
        HistoricTaskInstance historicTaskInstance = event.getHistoricTaskInstance();
        
        // 记录操作日志
        businessRelationService.recordOperation(
                historicTaskInstance.getProcessInstanceId(),
                historicTaskInstance.getId(),
                "TASK_COMPLETE",
                historicTaskInstance.getDeleteReason()
        );
    }
}
```

### 消息通知

#### 通知服务

```java
@Service
public class NotificationService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private WebSocketService webSocketService;

    /**
     * 发送任务通知
     */
    public void sendTaskNotification(String userId, String taskId, String taskName) {
        // 构建通知消息
        NotificationMessage message = new NotificationMessage();
        message.setType("TASK");
        message.setUserId(userId);
        message.setTitle("新任务提醒");
        message.setContent("您有一个新任务：" + taskName);
        message.setTaskId(taskId);
        message.setCreateTime(new Date());

        // 发送 WebSocket 实时通知
        webSocketService.sendNotification(userId, message);

        // 发送邮件通知（异步）
        sendEmailNotificationAsync(userId, message);

        // 发送到消息队列
        rabbitTemplate.convertAndSend("workflow.notifications", message);
    }

    /**
     * 发送任务分配通知
     */
    public void sendTaskAssignedNotification(String userId, String taskId, String taskName) {
        NotificationMessage message = new NotificationMessage();
        message.setType("TASK_ASSIGNED");
        message.setUserId(userId);
        message.setTitle("任务分配提醒");
        message.setContent("您被分配了一个任务：" + taskName);
        message.setTaskId(taskId);
        message.setCreateTime(new Date());

        webSocketService.sendNotification(userId, message);
        sendEmailNotificationAsync(userId, message);
    }

    /**
     * 发送流程完成通知
     */
    public void sendProcessCompleteNotification(String userId, String processInstanceId) {
        NotificationMessage message = new NotificationMessage();
        message.setType("PROCESS_COMPLETE");
        message.setUserId(userId);
        message.setTitle("流程完成通知");
        message.setContent("您发起的流程已完成");
        message.setProcessInstanceId(processInstanceId);
        message.setCreateTime(new Date());

        webSocketService.sendNotification(userId, message);
        sendEmailNotificationAsync(userId, message);
    }

    @Async
    public void sendEmailNotificationAsync(String userId, NotificationMessage message) {
        try {
            // 获取用户邮箱
            String email = getUserEmail(userId);
            if (email != null) {
                // 发送邮件
                sendEmail(email, message.getTitle(), message.getContent());
            }
        } catch (Exception e) {
            // 记录日志，不影响主流程
            log.error("发送邮件通知失败", e);
        }
    }

    private String getUserEmail(String userId) {
        // 从用户服务获取邮箱
        return userService.getUserEmail(userId);
    }

    private void sendEmail(String to, String subject, String content) {
        // 发送邮件
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }
}
```

#### WebSocket 服务

```java
@Service
public class WebSocketService {

    private final Map<String, WebSocketSession> sessions = new ConcurrentHashMap<>();

    /**
     * 发送通知
     */
    public void sendNotification(String userId, NotificationMessage message) {
        WebSocketSession session = sessions.get(userId);
        if (session != null && session.isOpen()) {
            try {
                session.sendMessage(new TextMessage(
                        objectMapper.writeValueAsString(message)));
            } catch (IOException e) {
                log.error("发送 WebSocket 通知失败", e);
            }
        }
    }

    /**
     * 注册会话
     */
    public void registerSession(String userId, WebSocketSession session) {
        sessions.put(userId, session);
    }

    /**
     * 移除会话
     */
    public void removeSession(String userId) {
        sessions.remove(userId);
    }
}
```

## 7.4 权限控制

### 流程权限

#### 流程权限注解

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ProcessPermission {

    /**
     * 权限类型
     */
    PermissionType type();

    /**
     * 流程定义Key（用于启动权限）
     */
    String processDefinitionKey() default "";

    /**
     * SpEL表达式，用于获取流程实例ID或任务ID
     */
    String value() default "";
}

public enum PermissionType {
    START,      // 启动流程权限
    VIEW,       // 查看流程权限
    COMPLETE,   // 完成任务权限
    WITHDRAW,   // 撤回流程权限
    DELEGATE,   // 委托任务权限
    TRANSFER    // 转办任务权限
}
```

#### 流程权限切面

```java
@Aspect
@Component
public class ProcessPermissionAspect {

    @Autowired
    private CustomPermissionHandler permissionHandler;

    @Around("@annotation(processPermission)")
    public Object checkPermission(ProceedingJoinPoint joinPoint, 
                                    ProcessPermission processPermission) throws Throwable {
        // 获取当前用户
        String userId = SecurityUtils.getCurrentUserId();

        // 获取权限类型
        PermissionType type = processPermission.type();

        // 根据权限类型进行校验
        switch (type) {
            case START:
                checkStartPermission(joinPoint, processPermission, userId);
                break;
            case VIEW:
                checkViewPermission(joinPoint, processPermission, userId);
                break;
            case COMPLETE:
                checkCompletePermission(joinPoint, processPermission, userId);
                break;
            case WITHDRAW:
                checkWithdrawPermission(joinPoint, processPermission, userId);
                break;
            case DELEGATE:
            case TRANSFER:
                checkDelegateOrTransferPermission(joinPoint, processPermission, userId);
                break;
            default:
                break;
        }

        return joinPoint.proceed();
    }

    private void checkStartPermission(ProceedingJoinPoint joinPoint, 
                                        ProcessPermission processPermission, 
                                        String userId) {
        String processDefinitionKey = processPermission.processDefinitionKey();
        if (processDefinitionKey.isEmpty()) {
            // 从参数中获取流程定义Key
            processDefinitionKey = getParameterValue(joinPoint, String.class);
        }

        if (!permissionHandler.canStartProcess(userId, processDefinitionKey)) {
            throw new BusinessException("没有启动该流程的权限");
        }
    }

    private void checkViewPermission(ProceedingJoinPoint joinPoint, 
                                       ProcessPermission processPermission, 
                                       String userId) {
        String processInstanceId = getProcessInstanceId(joinPoint, processPermission);
        if (!permissionHandler.canViewProcess(userId, processInstanceId)) {
            throw new BusinessException("没有查看该流程的权限");
        }
    }

    private void checkCompletePermission(ProceedingJoinPoint joinPoint, 
                                          ProcessPermission processPermission, 
                                          String userId) {
        String taskId = getTaskId(joinPoint, processPermission);
        if (!permissionHandler.canCompleteTask(userId, taskId)) {
            throw new BusinessException("没有办理该任务的权限");
        }
    }

    private void checkWithdrawPermission(ProceedingJoinPoint joinPoint, 
                                           ProcessPermission processPermission, 
                                           String userId) {
        String processInstanceId = getProcessInstanceId(joinPoint, processPermission);
        if (!permissionHandler.canWithdrawProcess(userId, processInstanceId)) {
            throw new BusinessException("没有撤回该流程的权限");
        }
    }

    private void checkDelegateOrTransferPermission(ProceedingJoinPoint joinPoint, 
                                                     ProcessPermission processPermission, 
                                                     String userId) {
        String taskId = getTaskId(joinPoint, processPermission);
        if (!permissionHandler.canDelegateOrTransferTask(userId, taskId)) {
            throw new BusinessException("没有委托或转办该任务的权限");
        }
    }

    private String getProcessInstanceId(ProceedingJoinPoint joinPoint, 
                                          ProcessPermission processPermission) {
        if (!processPermission.value().isEmpty()) {
            // 使用SpEL表达式获取
            return evaluateSpEL(joinPoint, processPermission.value(), String.class);
        }
        // 从参数中获取
        return getParameterValue(joinPoint, String.class, "processInstanceId");
    }

    private String getTaskId(ProceedingJoinPoint joinPoint, 
                              ProcessPermission processPermission) {
        if (!processPermission.value().isEmpty()) {
            return evaluateSpEL(joinPoint, processPermission.value(), String.class);
        }
        return getParameterValue(joinPoint, String.class, "taskId");
    }

    private <T> T getParameterValue(ProceedingJoinPoint joinPoint, Class<T> clazz) {
        Object[] args = joinPoint.getArgs();
        for (Object arg : args) {
            if (clazz.isInstance(arg)) {
                return clazz.cast(arg);
            }
        }
        throw new IllegalArgumentException("未找到参数: " + clazz.getName());
    }

    private <T> T getParameterValue(ProceedingJoinPoint joinPoint, Class<T> clazz, String paramName) {
        // 从方法签名中获取参数
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        String[] paramNames = signature.getParameterNames();
        Object[] args = joinPoint.getArgs();

        for (int i = 0; i < paramNames.length; i++) {
            if (paramName.equals(paramNames[i]) && clazz.isInstance(args[i])) {
                return clazz.cast(args[i]);
            }
        }
        throw new IllegalArgumentException("未找到参数: " + paramName);
    }

    private <T> T evaluateSpEL(ProceedingJoinPoint joinPoint, String expression, Class<T> clazz) {
        StandardEvaluationContext context = new StandardEvaluationContext();
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        String[] paramNames = signature.getParameterNames();
        Object[] args = joinPoint.getArgs();

        for (int i = 0; i < paramNames.length; i++) {
            context.setVariable(paramNames[i], args[i]);
        }

        ExpressionParser parser = new SpelExpressionParser();
        return parser.parseExpression(expression).getValue(context, clazz);
    }
}
```

#### 权限注解使用示例

```java
@RestController
@RequestMapping("/api/workflow")
public class WorkflowController {

    @PostMapping("/process/start")
    @ProcessPermission(type = PermissionType.START, processDefinitionKey = "#request.processDefinitionKey")
    public Result<ProcessInstanceDTO> startProcess(@RequestBody StartProcessRequest request) {
        // ...
    }

    @GetMapping("/process/{processInstanceId}")
    @ProcessPermission(type = PermissionType.VIEW, value = "#processInstanceId")
    public Result<ProcessInstanceDTO> getProcessInstance(@PathVariable String processInstanceId) {
        // ...
    }

    @PostMapping("/task/{taskId}/complete")
    @ProcessPermission(type = PermissionType.COMPLETE, value = "#taskId")
    public Result<Void> completeTask(@PathVariable String taskId, 
                                       @RequestBody CompleteTaskRequest request) {
        // ...
    }

    @PostMapping("/process/{processInstanceId}/withdraw")
    @ProcessPermission(type = PermissionType.WITHDRAW, value = "#processInstanceId")
    public Result<Void> withdrawProcess(@PathVariable String processInstanceId) {
        // ...
    }
}
```

### 数据权限

#### 数据权限过滤器

```java
@Component
public class DataPermissionFilter {

    @Autowired
    private DataPermissionService dataPermissionService;

    /**
     * 过滤流程实例列表
     */
    public List<ProcessInstance> filterProcessInstances(List<ProcessInstance> instances, String userId) {
        return instances.stream()
                .filter(instance -> hasDataPermission(userId, instance))
                .collect(Collectors.toList());
    }

    /**
     * 过滤任务列表
     */
    public List<Task> filterTasks(List<Task> tasks, String userId) {
        return tasks.stream()
                .filter(task -> hasDataPermission(userId, task))
                .collect(Collectors.toList());
    }

    private boolean hasDataPermission(String userId, ProcessInstance instance) {
        // 检查是否是发起人
        String startUserId = (String) runtimeService.getVariable(instance.getId(), "startUserId");
        if (userId.equals(startUserId)) {
            return true;
        }

        // 检查是否是流程管理员
        if (dataPermissionService.isProcessAdmin(userId)) {
            return true;
        }

        // 检查业务数据权限
        String businessKey = instance.getBusinessKey();
        return dataPermissionService.hasBusinessDataPermission(userId, businessKey);
    }

    private boolean hasDataPermission(String userId, Task task) {
        // 检查是否是办理人或候选人
        if (userId.equals(task.getAssignee())) {
            return true;
        }

        List<IdentityLink> identityLinks = taskService.getIdentityLinksForTask(task.getId());
        for (IdentityLink identityLink : identityLinks) {
            if (userId.equals(identityLink.getUserId())) {
                return true;
            }
            if (identityLink.getGroupId() != null) {
                if (isUserInGroup(userId, identityLink.getGroupId())) {
                    return true;
                }
            }
        }

        // 检查是否是流程管理员
        return dataPermissionService.isProcessAdmin(userId);
    }

    private boolean isUserInGroup(String userId, String groupId) {
        return identityService.createGroupQuery()
                .groupId(groupId)
                .groupMember(userId)
                .singleResult() != null;
    }
}
```

### 操作权限

#### 操作权限检查

```java
@Service
public class OperationPermissionService {

    @Autowired
    private TaskService taskService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private RuntimeService runtimeService;

    /**
     * 检查是否可以撤回流程
     */
    public boolean canWithdraw(String userId, String processInstanceId) {
        // 检查是否是发起人
        String startUserId = getStartUserId(processInstanceId);
        if (!userId.equals(startUserId)) {
            return false;
        }

        // 检查流程是否还在进行中
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .singleResult();
        if (processInstance == null) {
            return false;
        }

        // 检查是否只有一个任务且还未办理
        List<Task> tasks = taskService.createTaskQuery()
                .processInstanceId(processInstanceId)
                .list();
        if (tasks.size() != 1) {
            return false;
        }

        Task task = tasks.get(0);
        return task.getAssignee() == null;
    }

    /**
     * 检查是否可以加签
     */
    public boolean canAddSign(String userId, String taskId) {
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        if (task == null) {
            return false;
        }

        // 检查是否是当前办理人
        if (!userId.equals(task.getAssignee())) {
            return false;
        }

        // 检查任务是否支持加签
        return isTaskSupportAddSign(task);
    }

    /**
     * 检查是否可以退回
     */
    public boolean canReturn(String userId, String taskId) {
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        if (task == null) {
            return false;
        }

        // 检查是否是当前办理人
        if (!userId.equals(task.getAssignee())) {
            return false;
        }

        // 检查是否有上一个节点
        return hasPreviousActivity(task.getProcessInstanceId());
    }

    private String getStartUserId(String processInstanceId) {
        HistoricProcessInstance historicProcessInstance = historyService.createHistoricProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .singleResult();
        return historicProcessInstance != null ? historicProcessInstance.getStartUserId() : null;
    }

    private boolean isTaskSupportAddSign(Task task) {
        // 检查任务定义是否支持加签
        return true;
    }

    private boolean hasPreviousActivity(String processInstanceId) {
        List<HistoricActivityInstance> activities = historyService.createHistoricActivityInstanceQuery()
                .processInstanceId(processInstanceId)
                .finished()
                .orderByHistoricActivityInstanceEndTime()
                .desc()
                .list();
        return activities.size() > 1;
    }
}
```
