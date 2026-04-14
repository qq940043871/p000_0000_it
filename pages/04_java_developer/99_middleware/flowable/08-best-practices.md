# 最佳实践

## 8.1 性能优化

### 索引优化

#### 数据库索引设计

```sql
-- 流程实例表索引优化
CREATE INDEX idx_act_ru_execution_proc_inst ON ACT_RU_EXECUTION(PROC_INST_ID_);
CREATE INDEX idx_act_ru_execution_business_key ON ACT_RU_EXECUTION(BUSINESS_KEY_);
CREATE INDEX idx_act_ru_execution_proc_def_id ON ACT_RU_EXECUTION(PROC_DEF_ID_);

-- 任务表索引优化
CREATE INDEX idx_act_ru_task_proc_inst ON ACT_RU_TASK(PROC_INST_ID_);
CREATE INDEX idx_act_ru_task_execution ON ACT_RU_TASK(EXECUTION_ID_);
CREATE INDEX idx_act_ru_task_assignee ON ACT_RU_TASK(ASSIGNEE_);
CREATE INDEX idx_act_ru_task_create_time ON ACT_RU_TASK(CREATE_TIME_);

-- 历史表索引优化
CREATE INDEX idx_act_hi_procinst_start_time ON ACT_HI_PROCINST(START_TIME_);
CREATE INDEX idx_act_hi_procinst_end_time ON ACT_HI_PROCINST(END_TIME_);
CREATE INDEX idx_act_hi_procinst_proc_def_id ON ACT_HI_PROCINST(PROC_DEF_ID_);
CREATE INDEX idx_act_hi_taskinst_assignee ON ACT_HI_TASKINST(ASSIGNEE_);
CREATE INDEX idx_act_hi_taskinst_end_time ON ACT_HI_TASKINST(END_TIME_);
CREATE INDEX idx_act_hi_varinst_name ON ACT_HI_VARINST(NAME_);

-- 变量表索引优化
CREATE INDEX idx_act_ru_variable_execution ON ACT_RU_VARIABLE(EXECUTION_ID_);
CREATE INDEX idx_act_ru_variable_task ON ACT_RU_VARIABLE(TASK_ID_);
CREATE INDEX idx_act_ru_variable_name ON ACT_RU_VARIABLE(NAME_);
```

#### 自定义业务表索引

```sql
-- 流程业务关联表索引
CREATE INDEX idx_wf_business_relation_business_key ON wf_business_relation(business_key, business_type);
CREATE INDEX idx_wf_business_relation_start_user ON wf_business_relation(start_user_id);
CREATE INDEX idx_wf_business_relation_status ON wf_business_relation(status);
CREATE INDEX idx_wf_business_relation_start_time ON wf_business_relation(start_time);

-- 流程操作日志表索引
CREATE INDEX idx_wf_operation_log_process_instance ON wf_operation_log(process_instance_id);
CREATE INDEX idx_wf_operation_log_task ON wf_operation_log(task_id);
CREATE INDEX idx_wf_operation_log_operator ON wf_operation_log(operator_id);
CREATE INDEX idx_wf_operation_log_operation_time ON wf_operation_log(operation_time);
```

### 查询优化

#### 使用分页查询

```java
@Service
public class TaskQueryService {

    @Autowired
    private TaskService taskService;

    /**
     * 分页查询待办任务
     */
    public PageResult<TaskDTO> listTodoTasks(String userId, int pageNum, int pageSize) {
        // 使用分页查询，避免一次加载过多数据
        List<Task> tasks = taskService.createTaskQuery()
                .taskAssignee(userId)
                .active()
                .orderByTaskCreateTime()
                .desc()
                .listPage((pageNum - 1) * pageSize, pageSize);

        // 查询总数
        long total = taskService.createTaskQuery()
                .taskAssignee(userId)
                .active()
                .count();

        // 转换为DTO
        List<TaskDTO> taskDTOs = tasks.stream()
                .map(this::convertToDTO)
                .collect(Collectors.toList());

        return PageResult.of(taskDTOs, total, pageNum, pageSize);
    }

    /**
     * 批量查询流程变量
     */
    public Map<String, Map<String, Object>> getVariablesBatch(List<String> processInstanceIds) {
        Map<String, Map<String, Object>> result = new HashMap<>();
        
        // 批量查询，减少数据库访问次数
        for (String processInstanceId : processInstanceIds) {
            Map<String, Object> variables = runtimeService.getVariables(processInstanceId);
            result.put(processInstanceId, variables);
        }
        
        return result;
    }
}
```

#### 使用 Native 查询

```java
@Service
public class NativeQueryService {

    @Autowired
    private TaskService taskService;

    @Autowired
    private HistoryService historyService;

    /**
     * 使用 Native SQL 查询任务
     */
    public List<Task> listTasksByNativeQuery(String assignee) {
        return taskService.createNativeTaskQuery()
                .sql("SELECT * FROM ACT_RU_TASK WHERE ASSIGNEE_ = #{assignee} ORDER BY CREATE_TIME_ DESC")
                .parameter("assignee", assignee)
                .list();
    }

    /**
     * 使用 Native SQL 查询历史流程实例
     */
    public List<HistoricProcessInstance> listHistoricProcessInstancesByNativeQuery(
            String startUserId, Date startTime) {
        return historyService.createNativeHistoricProcessInstanceQuery()
                .sql("SELECT * FROM ACT_HI_PROCINST WHERE START_USER_ID_ = #{userId} AND START_TIME_ >= #{startTime}")
                .parameter("userId", startUserId)
                .parameter("startTime", startTime)
                .list();
    }

    /**
     * 复杂统计查询
     */
    public Map<String, Object> getTaskStatistics(String assignee) {
        String sql = "SELECT " +
                "COUNT(*) as totalCount, " +
                "SUM(CASE WHEN ASSIGNEE_ IS NULL THEN 1 ELSE 0 END) as unassignedCount, " +
                "SUM(CASE WHEN CREATE_TIME_ >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN 1 ELSE 0 END) as weekCount " +
                "FROM ACT_RU_TASK " +
                "WHERE ASSIGNEE_ = #{assignee}";

        List<Map<String, Object>> result = taskService.createNativeTaskQuery()
                .sql(sql)
                .parameter("assignee", assignee)
                .listMaps();

        return result.isEmpty() ? new HashMap<>() : result.get(0);
    }
}
```

### 缓存策略

#### Redis 缓存流程定义

```java
@Service
public class ProcessDefinitionCache {

    @Autowired
    private RepositoryService repositoryService;

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    private static final String PROCESS_DEFINITION_CACHE_KEY = "workflow:process:definition:";
    private static final long CACHE_EXPIRE_TIME = 3600; // 1小时

    /**
     * 获取流程定义（带缓存）
     */
    public ProcessDefinition getProcessDefinition(String processDefinitionKey) {
        String cacheKey = PROCESS_DEFINITION_CACHE_KEY + processDefinitionKey;
        
        // 先从缓存获取
        ProcessDefinition cached = (ProcessDefinition) redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            return cached;
        }
        
        // 从数据库查询
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .processDefinitionKey(processDefinitionKey)
                .latestVersion()
                .singleResult();
        
        // 写入缓存
        if (processDefinition != null) {
            redisTemplate.opsForValue().set(cacheKey, processDefinition, CACHE_EXPIRE_TIME, TimeUnit.SECONDS);
        }
        
        return processDefinition;
    }

    /**
     * 清除流程定义缓存
     */
    public void clearProcessDefinitionCache(String processDefinitionKey) {
        String cacheKey = PROCESS_DEFINITION_CACHE_KEY + processDefinitionKey;
        redisTemplate.delete(cacheKey);
    }

    /**
     * 清除所有流程定义缓存
     */
    public void clearAllProcessDefinitionCache() {
        Set<String> keys = redisTemplate.keys(PROCESS_DEFINITION_CACHE_KEY + "*");
        if (keys != null && !keys.isEmpty()) {
            redisTemplate.delete(keys);
        }
    }
}
```

#### 本地缓存用户信息

```java
@Service
public class UserCache {

    @Autowired
    private IdentityService identityService;

    // 使用 Guava Cache 作为本地缓存
    private final LoadingCache<String, User> userCache = CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .build(new CacheLoader<String, User>() {
                @Override
                public User load(String userId) {
                    return identityService.createUserQuery()
                            .userId(userId)
                            .singleResult();
                }
            });

    /**
     * 获取用户（带本地缓存）
     */
    public User getUser(String userId) {
        try {
            return userCache.get(userId);
        } catch (ExecutionException e) {
            return null;
        }
    }

    /**
     * 清除用户缓存
     */
    public void clearUserCache(String userId) {
        userCache.invalidate(userId);
    }

    /**
     * 清除所有用户缓存
     */
    public void clearAllUserCache() {
        userCache.invalidateAll();
    }
}
```

### 异步执行器优化

```java
@Configuration
public class AsyncExecutorConfig {

    @Bean
    public SpringProcessEngineConfiguration processEngineConfiguration(
            DataSource dataSource,
            PlatformTransactionManager transactionManager) {
        
        SpringProcessEngineConfiguration config = new SpringProcessEngineConfiguration();
        config.setDataSource(dataSource);
        config.setTransactionManager(transactionManager);
        config.setDatabaseSchemaUpdate("true");
        
        // 异步执行器配置
        config.setAsyncExecutorEnabled(true);
        config.setAsyncExecutorActivate(true);
        
        // 线程池配置
        config.setAsyncExecutorCorePoolSize(10);
        config.setAsyncExecutorMaxPoolSize(50);
        config.setAsyncExecutorQueueSize(100);
        
        // 作业执行配置
        config.setAsyncExecutorDefaultAsyncJobAcquireWaitTimeInMillis(1000);
        config.setAsyncExecutorDefaultTimerJobAcquireWaitTimeInMillis(1000);
        config.setAsyncExecutorTimerLockTimeInMillis(300000);
        config.setAsyncExecutorAsyncJobLockTimeInMillis(300000);
        
        // 重试配置
        config.setAsyncExecutorMaxAsyncJobsDuePerAcquisition(1);
        config.setAsyncExecutorMaxTimerJobsPerAcquisition(1);
        
        return config;
    }
}
```

## 8.2 安全考虑

### SQL 注入防护

#### 使用参数化查询

```java
@Service
public class SafeQueryService {

    @Autowired
    private TaskService taskService;

    /**
     * 安全的任务查询 - 使用 Flowable API
     */
    public List<Task> listTasksByAssignee(String assignee) {
        // 使用 Flowable 的查询 API，自动防止 SQL 注入
        return taskService.createTaskQuery()
                .taskAssignee(assignee)
                .list();
    }

    /**
     * 安全的 Native 查询 - 使用参数化
     */
    public List<Task> listTasksByNative(String assignee) {
        // 使用参数化查询，防止 SQL 注入
        return taskService.createNativeTaskQuery()
                .sql("SELECT * FROM ACT_RU_TASK WHERE ASSIGNEE_ = #{assignee}")
                .parameter("assignee", assignee)
                .list();
    }

    /**
     * 错误示例 - 不要使用字符串拼接
     */
    @Deprecated
    public List<Task> unsafeListTasksByNative(String assignee) {
        // 危险！容易受到 SQL 注入攻击
        String sql = "SELECT * FROM ACT_RU_TASK WHERE ASSIGNEE_ = '" + assignee + "'";
        return taskService.createNativeTaskQuery()
                .sql(sql)
                .list();
    }
}
```

### XSS 防护

#### 输入验证和输出编码

```java
@Service
public class XssProtectionService {

    /**
     * 清理用户输入
     */
    public String sanitizeInput(String input) {
        if (input == null) {
            return null;
        }
        
        // 使用 OWASP Java HTML Sanitizer
        PolicyFactory policy = Sanitizers.FORMATTING.and(Sanitizers.LINKS);
        return policy.sanitize(input);
    }

    /**
     * HTML 编码输出
     */
    public String encodeForHtml(String input) {
        if (input == null) {
            return null;
        }
        
        // 使用 Apache Commons Text
        return StringEscapeUtils.escapeHtml4(input);
    }

    /**
     * JavaScript 编码输出
     */
    public String encodeForJavaScript(String input) {
        if (input == null) {
            return null;
        }
        
        return StringEscapeUtils.escapeEcmaScript(input);
    }
}
```

#### Spring Boot XSS 防护配置

```java
@Configuration
public class WebSecurityConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 添加 XSS 防护拦截器
        registry.addInterceptor(new XssInterceptor())
                .addPathPatterns("/**");
    }
}

public class XssInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler) {
        // 包装请求，过滤 XSS
        XssHttpServletRequestWrapper wrappedRequest = 
            new XssHttpServletRequestWrapper(request);
        return true;
    }
}

public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {

    public XssHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (values == null) {
            return null;
        }
        
        String[] sanitizedValues = new String[values.length];
        for (int i = 0; i < values.length; i++) {
            sanitizedValues[i] = sanitize(values[i]);
        }
        return sanitizedValues;
    }

    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        return sanitize(value);
    }

    private String sanitize(String value) {
        if (value == null) {
            return null;
        }
        
        // 清理 XSS 字符
        value = value.replaceAll("<", "&lt;");
        value = value.replaceAll(">", "&gt;");
        value = value.replaceAll("\"", "&quot;");
        value = value.replaceAll("'", "&#x27;");
        value = value.replaceAll("/", "&#x2F;");
        
        return value;
    }
}
```

### 数据加密

#### 敏感变量加密存储

```java
@Service
public class EncryptedVariableService {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Autowired
    private EncryptionService encryptionService;

    /**
     * 设置加密的流程变量
     */
    public void setEncryptedVariable(String executionId, String name, String value) {
        // 加密敏感数据
        String encryptedValue = encryptionService.encrypt(value);
        runtimeService.setVariable(executionId, name + "_encrypted", encryptedValue);
    }

    /**
     * 获取解密的流程变量
     */
    public String getEncryptedVariable(String executionId, String name) {
        String encryptedValue = (String) runtimeService.getVariable(executionId, name + "_encrypted");
        if (encryptedValue == null) {
            return null;
        }
        return encryptionService.decrypt(encryptedValue);
    }

    /**
     * 设置加密的任务变量
     */
    public void setEncryptedTaskVariable(String taskId, String name, String value) {
        String encryptedValue = encryptionService.encrypt(value);
        taskService.setVariable(taskId, name + "_encrypted", encryptedValue);
    }

    /**
     * 获取解密的任务变量
     */
    public String getEncryptedTaskVariable(String taskId, String name) {
        String encryptedValue = (String) taskService.getVariable(taskId, name + "_encrypted");
        if (encryptedValue == null) {
            return null;
        }
        return encryptionService.decrypt(encryptedValue);
    }
}

@Service
public class EncryptionService {

    @Value("${encryption.secret-key}")
    private String secretKey;

    private static final String ALGORITHM = "AES";
    private static final String TRANSFORMATION = "AES/CBC/PKCS5Padding";

    /**
     * 加密
     */
    public String encrypt(String plainText) {
        try {
            SecretKeySpec keySpec = new SecretKeySpec(secretKey.getBytes(), ALGORITHM);
            IvParameterSpec ivSpec = new IvParameterSpec(new byte[16]);
            
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.ENCRYPT_MODE, keySpec, ivSpec);
            
            byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(encrypted);
        } catch (Exception e) {
            throw new RuntimeException("加密失败", e);
        }
    }

    /**
     * 解密
     */
    public String decrypt(String encryptedText) {
        try {
            SecretKeySpec keySpec = new SecretKeySpec(secretKey.getBytes(), ALGORITHM);
            IvParameterSpec ivSpec = new IvParameterSpec(new byte[16]);
            
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.DECRYPT_MODE, keySpec, ivSpec);
            
            byte[] decrypted = cipher.doFinal(Base64.getDecoder().decode(encryptedText));
            return new String(decrypted, StandardCharsets.UTF_8);
        } catch (Exception e) {
            throw new RuntimeException("解密失败", e);
        }
    }
}
```

## 8.3 监控运维

### 流程监控

#### 流程状态监控

```java
@Service
public class ProcessMonitorService {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private TaskService taskService;

    /**
     * 获取流程统计信息
     */
    public ProcessStatistics getProcessStatistics() {
        ProcessStatistics stats = new ProcessStatistics();
        
        // 活跃流程实例数
        stats.setActiveProcessInstanceCount(
                runtimeService.createProcessInstanceQuery().count());
        
        // 活跃任务数
        stats.setActiveTaskCount(
                taskService.createTaskQuery().active().count());
        
        // 今日完成流程数
        stats.setTodayCompletedCount(
                historyService.createHistoricProcessInstanceQuery()
                        .finished()
                        .finishedAfter(getTodayStart())
                        .count());
        
        // 今日启动流程数
        stats.setTodayStartedCount(
                historyService.createHistoricProcessInstanceQuery()
                        .startedAfter(getTodayStart())
                        .count());
        
        return stats;
    }

    /**
     * 获取超时任务列表
     */
    public List<TaskDTO> getOverdueTasks(int hours) {
        Date overdueTime = DateUtils.addHours(new Date(), -hours);
        
        List<Task> tasks = taskService.createTaskQuery()
                .active()
                .taskCreatedBefore(overdueTime)
                .orderByTaskCreateTime()
                .asc()
                .list();
        
        return tasks.stream()
                .map(this::convertToTaskDTO)
                .collect(Collectors.toList());
    }

    /**
     * 获取慢流程实例
     */
    public List<HistoricProcessInstanceDTO> getSlowProcessInstances(int hours) {
        Date threshold = DateUtils.addHours(new Date(), -hours);
        
        List<HistoricProcessInstance> instances = historyService.createHistoricProcessInstanceQuery()
                .unfinished()
                .startedBefore(threshold)
                .orderByProcessInstanceStartTime()
                .asc()
                .list();
        
        return instances.stream()
                .map(this::convertToHistoricProcessInstanceDTO)
                .collect(Collectors.toList());
    }

    private Date getTodayStart() {
        Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.HOUR_OF_DAY, 0);
        calendar.set(Calendar.MINUTE, 0);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);
        return calendar.getTime();
    }
}
```

#### 定时健康检查

```java
@Component
public class WorkflowHealthCheck {

    @Autowired
    private ProcessEngine processEngine;

    @Autowired
    private ManagementService managementService;

    @Scheduled(fixedRate = 60000) // 每分钟执行一次
    public void checkWorkflowHealth() {
        HealthCheckResult result = new HealthCheckResult();
        
        try {
            // 检查流程引擎
            result.setEngineHealthy(processEngine != null);
            
            // 检查数据库连接
            result.setDatabaseHealthy(checkDatabaseConnection());
            
            // 检查异步执行器
            result.setAsyncExecutorHealthy(checkAsyncExecutor());
            
            // 检查作业执行情况
            result.setJobHealthy(checkJobs());
            
            result.setStatus(result.isAllHealthy() ? "HEALTHY" : "UNHEALTHY");
            
            // 记录健康检查结果
            logHealthCheckResult(result);
            
        } catch (Exception e) {
            result.setStatus("ERROR");
            result.setErrorMessage(e.getMessage());
            log.error("工作流健康检查失败", e);
        }
    }

    private boolean checkDatabaseConnection() {
        try {
            // 执行简单查询测试数据库连接
            managementService.getTableName(Execution.class, false);
            return true;
        } catch (Exception e) {
            log.error("数据库连接检查失败", e);
            return false;
        }
    }

    private boolean checkAsyncExecutor() {
        try {
            ProcessEngineConfiguration config = processEngine.getProcessEngineConfiguration();
            AsyncExecutor asyncExecutor = config.getAsyncExecutor();
            return asyncExecutor != null && asyncExecutor.isActive();
        } catch (Exception e) {
            log.error("异步执行器检查失败", e);
            return false;
        }
    }

    private boolean checkJobs() {
        try {
            // 检查是否有死信作业
            long deadLetterJobCount = managementService.createDeadLetterJobQuery().count();
            if (deadLetterJobCount > 0) {
                log.warn("发现 {} 个死信作业", deadLetterJobCount);
            }
            return deadLetterJobCount == 0;
        } catch (Exception e) {
            log.error("作业检查失败", e);
            return false;
        }
    }

    private void logHealthCheckResult(HealthCheckResult result) {
        if ("UNHEALTHY".equals(result.getStatus()) || "ERROR".equals(result.getStatus())) {
            // 发送告警
            sendAlert(result);
        }
        // 记录日志
        log.info("工作流健康检查结果: {}", result);
    }

    private void sendAlert(HealthCheckResult result) {
        // 发送告警通知（邮件、短信、钉钉等）
        log.warn("发送工作流告警: {}", result);
    }
}
```

### 日志记录

#### 流程操作日志

```java
@Aspect
@Component
public class WorkflowLogAspect {

    private static final Logger log = LoggerFactory.getLogger(WorkflowLogAspect.class);

    @Autowired
    private OperationLogService operationLogService;

    @Pointcut("@annotation(com.example.workflow.annotation.WorkflowLog)")
    public void workflowLogPointcut() {
    }

    @Around("workflowLogPointcut()")
    public Object logWorkflowOperation(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        WorkflowLog workflowLog = method.getAnnotation(WorkflowLog.class);
        
        String operation = workflowLog.operation();
        String description = workflowLog.description();
        String operator = SecurityUtils.getCurrentUserId();
        
        long startTime = System.currentTimeMillis();
        Object result = null;
        Exception exception = null;
        
        try {
            result = joinPoint.proceed();
            return result;
        } catch (Exception e) {
            exception = e;
            throw e;
        } finally {
            long duration = System.currentTimeMillis() - startTime;
            boolean success = exception == null;
            
            // 记录操作日志
            OperationLog logEntry = new OperationLog();
            logEntry.setOperation(operation);
            logEntry.setDescription(description);
            logEntry.setOperator(operator);
            logEntry.setDuration(duration);
            logEntry.setSuccess(success);
            logEntry.setErrorMessage(exception != null ? exception.getMessage() : null);
            logEntry.setCreateTime(new Date());
            
            // 获取流程实例ID和任务ID
            Object[] args = joinPoint.getArgs();
            for (Object arg : args) {
                if (arg instanceof String) {
                    String argStr = (String) arg;
                    if (argStr.startsWith("proc-")) {
                        logEntry.setProcessInstanceId(argStr);
                    } else if (argStr.startsWith("task-")) {
                        logEntry.setTaskId(argStr);
                    }
                }
            }
            
            operationLogService.save(logEntry);
            
            // 记录日志
            log.info("工作流操作: {}, 操作人: {}, 耗时: {}ms, 成功: {}", 
                    operation, operator, duration, success);
        }
    }
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface WorkflowLog {

    /**
     * 操作类型
     */
    String operation();

    /**
     * 操作描述
     */
    String description() default "";
}
```

### 故障排查

#### 常见问题排查

```java
@Service
public class TroubleshootingService {

    @Autowired
    private ManagementService managementService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private RuntimeService runtimeService;

    /**
     * 检查死信作业
     */
    public List<DeadLetterJob> getDeadLetterJobs() {
        return managementService.createDeadLetterJobQuery()
                .orderByJobDuedate()
                .desc()
                .list();
    }

    /**
     * 重试死信作业
     */
    public void retryDeadLetterJob(String jobId) {
        DeadLetterJob job = managementService.createDeadLetterJobQuery()
                .deadLetterJobId(jobId)
                .singleResult();
        
        if (job != null) {
            // 移动到可执行作业
            managementService.moveDeadLetterJobToExecutableJob(jobId, 3);
            log.info("已重试死信作业: {}", jobId);
        }
    }

    /**
     * 删除死信作业
     */
    public void deleteDeadLetterJob(String jobId) {
        managementService.deleteDeadLetterJob(jobId);
        log.info("已删除死信作业: {}", jobId);
    }

    /**
     * 获取流程执行历史
     */
    public List<ActivityInstanceHistory> getActivityInstanceHistory(String processInstanceId) {
        List<HistoricActivityInstance> activities = historyService.createHistoricActivityInstanceQuery()
                .processInstanceId(processInstanceId)
                .orderByHistoricActivityInstanceStartTime()
                .asc()
                .list();
        
        return activities.stream()
                .map(this::convertToActivityInstanceHistory)
                .collect(Collectors.toList());
    }

    /**
     * 检查流程实例状态
     */
    public ProcessInstanceStatus checkProcessInstanceStatus(String processInstanceId) {
        ProcessInstanceStatus status = new ProcessInstanceStatus();
        
        // 检查流程实例是否存在
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .singleResult();
        
        if (processInstance == null) {
            // 检查历史记录
            HistoricProcessInstance historicProcessInstance = historyService.createHistoricProcessInstanceQuery()
                    .processInstanceId(processInstanceId)
                    .singleResult();
            
            if (historicProcessInstance == null) {
                status.setStatus("NOT_FOUND");
                status.setMessage("流程实例不存在");
            } else {
                status.setStatus("COMPLETED");
                status.setMessage("流程已完成");
                status.setEndTime(historicProcessInstance.getEndTime());
                status.setDuration(historicProcessInstance.getDurationInMillis());
            }
        } else {
            status.setStatus("ACTIVE");
            status.setMessage("流程进行中");
            
            // 获取当前任务
            List<Task> tasks = runtimeService.getTaskService().createTaskQuery()
                    .processInstanceId(processInstanceId)
                    .list();
            status.setCurrentTasks(tasks);
            
            // 获取当前执行
            List<Execution> executions = runtimeService.createExecutionQuery()
                    .processInstanceId(processInstanceId)
                    .list();
            status.setCurrentExecutions(executions);
        }
        
        return status;
    }

    /**
     * 强制终止流程实例
     */
    public void forceTerminateProcessInstance(String processInstanceId, String reason) {
        runtimeService.deleteProcessInstance(processInstanceId, reason);
        log.info("已强制终止流程实例: {}, 原因: {}", processInstanceId, reason);
    }
}
```

## 8.4 测试策略

### 单元测试

#### 流程定义测试

```java
@SpringBootTest
public class ProcessDefinitionTest {

    @Autowired
    private RepositoryService repositoryService;

    @Test
    public void testProcessDefinitionDeployment() {
        // 部署流程定义
        Deployment deployment = repositoryService.createDeployment()
                .addClasspathResource("processes/leave-approval.bpmn20.xml")
                .deploy();
        
        // 验证部署成功
        assertNotNull(deployment);
        assertNotNull(deployment.getId());
        
        // 验证流程定义存在
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .deploymentId(deployment.getId())
                .singleResult();
        
        assertNotNull(processDefinition);
        assertEquals("leave-approval", processDefinition.getKey());
        
        // 清理
        repositoryService.deleteDeployment(deployment.getId(), true);
    }

    @Test
    public void testProcessDefinitionVersion() {
        // 第一次部署
        Deployment deployment1 = repositoryService.createDeployment()
                .addClasspathResource("processes/leave-approval.bpmn20.xml")
                .deploy();
        
        // 第二次部署
        Deployment deployment2 = repositoryService.createDeployment()
                .addClasspathResource("processes/leave-approval.bpmn20.xml")
                .deploy();
        
        // 验证版本
        ProcessDefinition pd1 = repositoryService.createProcessDefinitionQuery()
                .deploymentId(deployment1.getId())
                .singleResult();
        
        ProcessDefinition pd2 = repositoryService.createProcessDefinitionQuery()
                .deploymentId(deployment2.getId())
                .singleResult();
        
        assertEquals(1, pd1.getVersion());
        assertEquals(2, pd2.getVersion());
        
        // 清理
        repositoryService.deleteDeployment(deployment1.getId(), true);
        repositoryService.deleteDeployment(deployment2.getId(), true);
    }
}
```

#### 流程执行测试

```java
@SpringBootTest
public class ProcessExecutionTest {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private IdentityService identityService;

    @Test
    public void testCompleteProcessExecution() {
        // 设置当前用户
        identityService.setAuthenticatedUserId("test-user");
        
        // 启动流程
        Map<String, Object> variables = new HashMap<>();
        variables.put("assignee", "manager");
        variables.put("days", 2);
        
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                "leave-approval", variables);
        
        assertNotNull(processInstance);
        assertNotNull(processInstance.getId());
        
        // 验证第一个任务
        Task task = taskService.createTaskQuery()
                .processInstanceId(processInstance.getId())
                .singleResult();
        
        assertNotNull(task);
        assertEquals("manager", task.getAssignee());
        
        // 完成任务
        taskService.complete(task.getId());
        
        // 验证流程完成
        HistoricProcessInstance historicProcessInstance = historyService.createHistoricProcessInstanceQuery()
                .processInstanceId(processInstance.getId())
                .singleResult();
        
        assertNotNull(historicProcessInstance);
        assertNotNull(historicProcessInstance.getEndTime());
    }

    @Test
    public void testProcessVariables() {
        // 启动流程
        Map<String, Object> variables = new HashMap<>();
        variables.put("stringVar", "test");
        variables.put("intVar", 100);
        variables.put("booleanVar", true);
        
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                "leave-approval", variables);
        
        // 获取变量
        String stringVar = (String) runtimeService.getVariable(processInstance.getId(), "stringVar");
        Integer intVar = (Integer) runtimeService.getVariable(processInstance.getId(), "intVar");
        Boolean booleanVar = (Boolean) runtimeService.getVariable(processInstance.getId(), "booleanVar");
        
        // 验证变量
        assertEquals("test", stringVar);
        assertEquals(Integer.valueOf(100), intVar);
        assertTrue(booleanVar);
        
        // 更新变量
        runtimeService.setVariable(processInstance.getId(), "stringVar", "updated");
        String updatedVar = (String) runtimeService.getVariable(processInstance.getId(), "stringVar");
        assertEquals("updated", updatedVar);
    }

    @Test
    public void testTaskAssignment() {
        // 启动流程
        Map<String, Object> variables = new HashMap<>();
        variables.put("assigneeList", Arrays.asList("user1", "user2", "user3"));
        
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                "countersign-process", variables);
        
        // 验证多实例任务
        List<Task> tasks = taskService.createTaskQuery()
                .processInstanceId(processInstance.getId())
                .list();
        
        assertEquals(3, tasks.size());
        
        // 完成任务
        for (Task task : tasks) {
            taskService.complete(task.getId());
        }
        
        // 验证流程继续
        Task nextTask = taskService.createTaskQuery()
                .processInstanceId(processInstance.getId())
                .singleResult();
        
        assertNotNull(nextTask);
    }
}
```

### 集成测试

#### 完整流程测试

```java
@SpringBootTest
@AutoConfigureMockMvc
public class WorkflowIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private WorkflowService workflowService;

    @Test
    @WithMockUser(username = "test-user")
    public void testStartProcessViaApi() throws Exception {
        // 准备请求
        StartProcessRequest request = new StartProcessRequest();
        request.setProcessDefinitionKey("leave-approval");
        request.setBusinessKey("BUS-001");
        
        Map<String, Object> variables = new HashMap<>();
        variables.put("assignee", "manager");
        variables.put("days", 2);
        variables.put("reason", "家里有事");
        request.setVariables(variables);
        
        // 发送请求
        mockMvc.perform(post("/api/workflow/process/start")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data.processInstanceId").exists());
    }

    @Test
    @WithMockUser(username = "manager")
    public void testCompleteTaskViaApi() throws Exception {
        // 先启动流程
        ProcessInstanceDTO processInstance = workflowService.startProcess(
                "leave-approval", "BUS-002", 
                Collections.singletonMap("assignee", "manager"));
        
        // 获取任务
        PageResult<TaskDTO> tasks = workflowService.listTodoTasks(
                "manager", new PageQuery(1, 10));
        TaskDTO task = tasks.getList().get(0);
        
        // 准备完成任务请求
        CompleteTaskRequest request = new CompleteTaskRequest();
        Map<String, Object> variables = new HashMap<>();
        variables.put("approved", true);
        variables.put("comment", "同意");
        request.setVariables(variables);
        
        // 发送请求
        mockMvc.perform(post("/api/workflow/task/{taskId}/complete", task.getId())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200));
    }

    @Test
    @WithMockUser(username = "test-user")
    public void testGetTodoTasksViaApi() throws Exception {
        mockMvc.perform(get("/api/workflow/task/todo")
                        .param("pageNum", "1")
                        .param("pageSize", "10"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data.list").exists());
    }

    @Test
    @WithMockUser(username = "test-user")
    public void testGetProcessInstanceViaApi() throws Exception {
        // 先启动流程
        ProcessInstanceDTO processInstance = workflowService.startProcess(
                "leave-approval", "BUS-003", 
                Collections.singletonMap("assignee", "manager"));
        
        // 查询流程实例
        mockMvc.perform(get("/api/workflow/process/{processInstanceId}", 
                        processInstance.getProcessInstanceId()))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data.processInstanceId").value(processInstance.getProcessInstanceId()));
    }
}
```

### 性能测试

#### 流程启动性能测试

```java
@SpringBootTest
public class WorkflowPerformanceTest {

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private TaskService taskService;

    @Test
    public void testProcessStartupPerformance() {
        int iterations = 1000;
        List<Long> durations = new ArrayList<>();
        
        for (int i = 0; i < iterations; i++) {
            long startTime = System.currentTimeMillis();
            
            // 启动流程
            ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                    "leave-approval",
                    Collections.singletonMap("assignee", "user" + i));
            
            // 立即删除
            runtimeService.deleteProcessInstance(processInstance.getId(), "test-cleanup");
            
            long duration = System.currentTimeMillis() - startTime;
            durations.add(duration);
        }
        
        // 计算统计数据
        long totalTime = durations.stream().mapToLong(Long::longValue).sum();
        double avgTime = (double) totalTime / iterations;
        long maxTime = durations.stream().mapToLong(Long::longValue).max().orElse(0);
        long minTime = durations.stream().mapToLong(Long::longValue).min().orElse(0);
        
        System.out.println("流程启动性能测试结果:");
        System.out.println("总次数: " + iterations);
        System.out.println("总耗时: " + totalTime + "ms");
        System.out.println("平均耗时: " + String.format("%.2f", avgTime) + "ms");
        System.out.println("最大耗时: " + maxTime + "ms");
        System.out.println("最小耗时: " + minTime + "ms");
        
        // 验证性能指标
        assertTrue(avgTime < 100, "平均耗时应该小于100ms");
    }

    @Test
    public void testConcurrentProcessStartup() throws InterruptedException {
        int threadCount = 10;
        int processesPerThread = 100;
        CountDownLatch latch = new CountDownLatch(threadCount);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger failCount = new AtomicInteger(0);
        
        ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
        
        for (int i = 0; i < threadCount; i++) {
            final int threadIndex = i;
            executorService.submit(() -> {
                try {
                    for (int j = 0; j < processesPerThread; j++) {
                        try {
                            ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                                    "leave-approval",
                                    Collections.singletonMap("assignee", "user-" + threadIndex + "-" + j));
                            runtimeService.deleteProcessInstance(processInstance.getId(), "cleanup");
                            successCount.incrementAndGet();
                        } catch (Exception e) {
                            failCount.incrementAndGet();
                        }
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        latch.await();
        executorService.shutdown();
        
        System.out.println("并发流程启动测试结果:");
        System.out.println("成功次数: " + successCount.get());
        System.out.println("失败次数: " + failCount.get());
        System.out.println("成功率: " + 
                String.format("%.2f%%", (double) successCount.get() / (threadCount * processesPerThread) * 100));
        
        // 验证成功率
        assertEquals(0, failCount.get(), "不应该有失败的流程启动");
    }

    @Test
    public void testTaskQueryPerformance() {
        // 先创建大量任务
        int taskCount = 10000;
        for (int i = 0; i < taskCount; i++) {
            ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(
                    "leave-approval",
                    Collections.singletonMap("assignee", "test-user"));
        }
        
        // 测试查询性能
        long startTime = System.currentTimeMillis();
        List<Task> tasks = taskService.createTaskQuery()
                .taskAssignee("test-user")
                .orderByTaskCreateTime()
                .desc()
                .listPage(0, 100);
        long queryTime = System.currentTimeMillis() - startTime;
        
        System.out.println("任务查询性能测试:");
        System.out.println("查询任务数: 100");
        System.out.println("总任务数: " + taskCount);
        System.out.println("查询耗时: " + queryTime + "ms");
        
        // 验证查询性能
        assertTrue(queryTime < 500, "查询耗时应该小于500ms");
        
        // 清理
        List<ProcessInstance> instances = runtimeService.createProcessInstanceQuery().list();
        for (ProcessInstance instance : instances) {
            runtimeService.deleteProcessInstance(instance.getId(), "cleanup");
        }
    }
}
```
