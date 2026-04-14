# FlyFlow 改造

## 6.1 FlyFlow 介绍

### 项目背景

FlyFlow 是一个仿照飞书审批的开源工作流项目，提供了现代化的流程设计和管理界面。

**核心特性：**
- 现代化的 UI 设计，类似飞书审批
- 拖拽式流程设计器
- 丰富的表单组件
- 灵活的审批人配置
- 移动端适配
- 实时流程监控

### 技术架构

```
FlyFlow 技术架构
├── 前端
│   ├── Vue 3 + TypeScript
│   ├── Vite 构建工具
│   ├── Element Plus UI 框架
│   └── bpmn-js 流程设计器
├── 后端
│   ├── Spring Boot 2.7+
│   ├── Flowable 6.8+
│   ├── MyBatis Plus
│   └── MySQL 8.0+
└── 基础设施
    ├── Redis 缓存
    ├── RabbitMQ 消息队列
    └── MinIO 文件存储
```

### 主要功能模块

| 模块 | 功能说明 |
|------|----------|
| 流程设计器 | 拖拽式流程设计，支持 BPMN 2.0 |
| 表单设计器 | 可视化表单设计，丰富的组件库 |
| 流程管理 | 流程部署、版本管理、启动/停止 |
| 待办中心 | 个人待办、已办、抄送任务 |
| 流程监控 | 实时流程图、进度追踪 |
| 统计分析 | 流程效率、审批时长统计 |

## 6.2 核心改造点

### UI/UX 优化

#### 主题定制

```javascript
// src/theme/index.js
export default {
  token: {
    colorPrimary: '#1890ff',
    colorSuccess: '#52c41a',
    colorWarning: '#faad14',
    colorError: '#ff4d4f',
    borderRadius: 6,
    fontSize: 14
  },
  components: {
    Button: {
      borderRadius: 4,
      controlHeight: 36
    },
    Input: {
      borderRadius: 4,
      controlHeight: 36
    }
  }
}
```

#### 响应式布局

```vue
<template>
  <div class="workflow-container">
    <el-container>
      <el-aside :width="sidebarWidth" class="sidebar">
        <Sidebar />
      </el-aside>
      <el-main class="main-content">
        <router-view />
      </el-main>
    </el-container>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useStore } from 'vuex'

const store = useStore()
const sidebarWidth = computed(() => store.state.app.sidebarCollapsed ? '64px' : '240px')
</script>

<style scoped>
.workflow-container {
  height: 100vh;
  display: flex;
}

.sidebar {
  transition: width 0.3s ease;
  overflow: hidden;
}

.main-content {
  flex: 1;
  overflow: auto;
  background-color: #f5f7fa;
}

@media (max-width: 768px) {
  .sidebar {
    position: fixed;
    left: -240px;
    z-index: 1000;
  }
  
  .sidebar.mobile-open {
    left: 0;
  }
}
</style>
```

### 流程设计器改造

#### 自定义工具栏

```javascript
// src/components/ProcessDesigner/toolbar.js
export const customToolbar = [
  {
    type: 'button',
    icon: 'save',
    title: '保存',
    action: 'save'
  },
  {
    type: 'button',
    icon: 'preview',
    title: '预览',
    action: 'preview'
  },
  {
    type: 'divider'
  },
  {
    type: 'button',
    icon: 'undo',
    title: '撤销',
    action: 'undo'
  },
  {
    type: 'button',
    icon: 'redo',
    title: '重做',
    action: 'redo'
  },
  {
    type: 'divider'
  },
  {
    type: 'button',
    icon: 'zoom-in',
    title: '放大',
    action: 'zoomIn'
  },
  {
    type: 'button',
    icon: 'zoom-out',
    title: '缩小',
    action: 'zoomOut'
  },
  {
    type: 'button',
    icon: 'zoom-reset',
    title: '重置',
    action: 'zoomReset'
  }
]
```

#### 自定义节点面板

```vue
<template>
  <div class="custom-palette">
    <div class="palette-header">
      <h4>节点组件</h4>
    </div>
    <div class="palette-items">
      <div 
        v-for="item in paletteItems" 
        :key="item.type"
        class="palette-item"
        draggable="true"
        @dragstart="onDragStart($event, item)"
      >
        <el-icon :size="20"><component :is="item.icon" /></el-icon>
        <span>{{ item.label }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { 
  VideoPlay, 
  Timer, 
  User, 
  Service, 
  Decision, 
  Parallel, 
  SubProcess 
} from '@element-plus/icons-vue'

const paletteItems = ref([
  { type: 'bpmn:StartEvent', label: '开始', icon: VideoPlay },
  { type: 'bpmn:EndEvent', label: '结束', icon: VideoPlay },
  { type: 'bpmn:UserTask', label: '用户任务', icon: User },
  { type: 'bpmn:ServiceTask', label: '服务任务', icon: Service },
  { type: 'bpmn:ExclusiveGateway', label: '排他网关', icon: Decision },
  { type: 'bpmn:ParallelGateway', label: '并行网关', icon: Parallel },
  { type: 'bpmn:SubProcess', label: '子流程', icon: SubProcess },
  { type: 'bpmn:TimerEventDefinition', label: '定时器', icon: Timer }
])

const onDragStart = (event, item) => {
  event.dataTransfer.setData('application/bpmn', JSON.stringify(item))
}
</script>

<style scoped>
.custom-palette {
  background: white;
  border-right: 1px solid #e4e7ed;
  width: 200px;
  height: 100%;
}

.palette-header {
  padding: 16px;
  border-bottom: 1px solid #e4e7ed;
  background: #f5f7fa;
}

.palette-items {
  padding: 8px;
}

.palette-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  margin-bottom: 4px;
  border-radius: 4px;
  cursor: grab;
  transition: all 0.2s;
}

.palette-item:hover {
  background: #ecf5ff;
  color: #409eff;
}
</style>
```

### 移动端适配

#### 触摸手势支持

```javascript
// src/utils/touch.js
export class TouchGesture {
  constructor(element) {
    this.element = element
    this.startX = 0
    this.startY = 0
    this.currentX = 0
    this.currentY = 0
    this.init()
  }

  init() {
    this.element.addEventListener('touchstart', this.onTouchStart.bind(this), { passive: false })
    this.element.addEventListener('touchmove', this.onTouchMove.bind(this), { passive: false })
    this.element.addEventListener('touchend', this.onTouchEnd.bind(this))
  }

  onTouchStart(event) {
    const touch = event.touches[0]
    this.startX = touch.clientX
    this.startY = touch.clientY
    this.currentX = touch.clientX
    this.currentY = touch.clientY
  }

  onTouchMove(event) {
    event.preventDefault()
    const touch = event.touches[0]
    this.currentX = touch.clientX
    this.currentY = touch.clientY
    
    const deltaX = this.currentX - this.startX
    const deltaY = this.currentY - this.startY
    
    this.emit('pan', { deltaX, deltaY })
  }

  onTouchEnd() {
    const deltaX = this.currentX - this.startX
    const deltaY = this.currentY - this.startY
    
    if (Math.abs(deltaX) > 50 && Math.abs(deltaX) > Math.abs(deltaY)) {
      this.emit(deltaX > 0 ? 'swipeRight' : 'swipeLeft')
    }
  }

  emit(eventName, data) {
    const event = new CustomEvent(eventName, { detail: data })
    this.element.dispatchEvent(event)
  }
}
```

#### 移动端任务列表

```vue
<template>
  <div class="mobile-task-list">
    <div class="task-header">
      <div class="tabs">
        <div 
          v-for="tab in tabs" 
          :key="tab.key"
          :class="['tab-item', { active: activeTab === tab.key }]"
          @click="activeTab = tab.key"
        >
          {{ tab.label }}
        </div>
      </div>
    </div>
    
    <div class="task-content">
      <div v-for="task in filteredTasks" :key="task.id" class="task-card">
        <div class="task-card-header">
          <div class="task-title">{{ task.name }}</div>
          <div class="task-priority" :class="'priority-' + task.priority">
            {{ getPriorityText(task.priority) }}
          </div>
        </div>
        <div class="task-card-body">
          <div class="task-info">
            <el-icon><User /></el-icon>
            <span>{{ task.assignee }}</span>
          </div>
          <div class="task-info">
            <el-icon><Clock /></el-icon>
            <span>{{ formatDate(task.createTime) }}</span>
          </div>
        </div>
        <div class="task-card-footer">
          <el-button type="primary" size="small" @click="handleTask(task)">
            办理
          </el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { User, Clock } from '@element-plus/icons-vue'
import dayjs from 'dayjs'

const tabs = ref([
  { key: 'todo', label: '待办' },
  { key: 'done', label: '已办' },
  { key: 'cc', label: '抄送' }
])

const activeTab = ref('todo')
const tasks = ref([])

const filteredTasks = computed(() => {
  return tasks.value.filter(task => task.status === activeTab.value)
})

const getPriorityText = (priority) => {
  const map = { 1: '紧急', 2: '高', 3: '中', 4: '低' }
  return map[priority] || '普通'
}

const formatDate = (date) => {
  return dayjs(date).format('MM-DD HH:mm')
}

const handleTask = (task) => {
  // 处理任务
}
</script>

<style scoped>
.mobile-task-list {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f7fa;
}

.task-header {
  background: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.tabs {
  display: flex;
  padding: 0 16px;
}

.tab-item {
  flex: 1;
  text-align: center;
  padding: 16px 0;
  font-size: 15px;
  color: #606266;
  border-bottom: 2px solid transparent;
  transition: all 0.3s;
}

.tab-item.active {
  color: #409eff;
  border-bottom-color: #409eff;
  font-weight: 500;
}

.task-content {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

.task-card {
  background: white;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 12px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
}

.task-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.task-title {
  font-size: 16px;
  font-weight: 500;
  color: #303133;
}

.task-priority {
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 4px;
}

.priority-1 {
  background: #fef0f0;
  color: #f56c6c;
}

.priority-2 {
  background: #fdf6ec;
  color: #e6a23c;
}

.task-card-body {
  margin-bottom: 12px;
}

.task-info {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: #909399;
  margin-bottom: 6px;
}

.task-card-footer {
  display: flex;
  justify-content: flex-end;
}
</style>
```

## 6.3 与业务系统集成

### 单点登录

#### OAuth2.0 集成

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private OAuth2UserService<OAuth2UserRequest, OAuth2User> oAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/login", "/oauth2/**", "/public/**").permitAll()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .loginPage("/login")
                .userInfoEndpoint()
                    .userService(oAuth2UserService)
                    .and()
                .successHandler(oAuth2AuthenticationSuccessHandler())
                .and()
            .logout()
                .logoutSuccessUrl("/login")
                .invalidateHttpSession(true)
                .clearAuthentication(true);
    }

    @Bean
    public AuthenticationSuccessHandler oAuth2AuthenticationSuccessHandler() {
        return (request, response, authentication) -> {
            OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();
            
            // 同步用户信息
            String userId = oAuth2User.getAttribute("sub");
            String userName = oAuth2User.getAttribute("name");
            String email = oAuth2User.getAttribute("email");
            
            syncUser(userId, userName, email);
            
            // 生成 JWT Token
            String token = jwtTokenProvider.generateToken(userId);
            
            // 重定向到首页
            response.sendRedirect("/?token=" + token);
        };
    }

    private void syncUser(String userId, String userName, String email) {
        IdentityService identityService = processEngine.getIdentityService();
        
        User user = identityService.createUserQuery()
                .userId(userId)
                .singleResult();
        
        if (user == null) {
            user = identityService.newUser(userId);
        }
        
        user.setFirstName(userName);
        user.setEmail(email);
        identityService.saveUser(user);
    }
}
```

#### JWT Token 验证

```java
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration}")
    private int jwtExpiration;

    public String generateToken(String userId) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpiration);

        return Jwts.builder()
                .setSubject(userId)
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();
    }

    public String getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(jwtSecret)
                .parseClaimsJws(token)
                .getBody();
        
        return claims.getSubject();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (SignatureException | MalformedJwtException | ExpiredJwtException 
                | UnsupportedJwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

### 数据同步

#### 用户数据同步

```java
@Service
public class UserSyncService {

    @Autowired
    private IdentityService identityService;

    @Autowired
    private BusinessUserApi businessUserApi;

    /**
     * 全量同步用户
     */
    @Scheduled(cron = "0 0 2 * * ?")
    public void syncAllUsers() {
        List<BusinessUser> businessUsers = businessUserApi.getAllUsers();
        
        for (BusinessUser businessUser : businessUsers) {
            syncUser(businessUser);
        }
    }

    /**
     * 同步单个用户
     */
    @Async
    public void syncUser(BusinessUser businessUser) {
        User user = identityService.createUserQuery()
                .userId(businessUser.getId())
                .singleResult();
        
        if (user == null) {
            user = identityService.newUser(businessUser.getId());
        }
        
        user.setFirstName(businessUser.getName());
        user.setEmail(businessUser.getEmail());
        identityService.saveUser(user);
        
        // 同步用户组
        syncUserGroups(businessUser);
    }

    /**
     * 同步用户组
     */
    private void syncUserGroups(BusinessUser businessUser) {
        List<String> groupIds = businessUser.getGroupIds();
        
        for (String groupId : groupIds) {
            Group group = identityService.createGroupQuery()
                    .groupId(groupId)
                    .singleResult();
            
            if (group == null) {
                group = identityService.newGroup(groupId);
                group.setName(businessUserApi.getGroupName(groupId));
                group.setType("assignment");
                identityService.saveGroup(group);
            }
            
            // 检查用户是否已在组中
            boolean isMember = identityService.createGroupQuery()
                    .groupId(groupId)
                    .groupMember(businessUser.getId())
                    .singleResult() != null;
            
            if (!isMember) {
                identityService.createMembership(businessUser.getId(), groupId);
            }
        }
    }
}
```

#### 组织架构同步

```java
@Service
public class OrganizationSyncService {

    @Autowired
    private IdentityService identityService;

    @Autowired
    private OrganizationApi organizationApi;

    /**
     * 同步组织架构
     */
    @Scheduled(cron = "0 0 3 * * ?")
    public void syncOrganization() {
        List<Organization> organizations = organizationApi.getAllOrganizations();
        
        for (Organization org : organizations) {
            syncOrganization(org);
        }
    }

    /**
     * 同步单个组织
     */
    private void syncOrganization(Organization org) {
        Group group = identityService.createGroupQuery()
                .groupId(org.getId())
                .singleResult();
        
        if (group == null) {
            group = identityService.newGroup(org.getId());
        }
        
        group.setName(org.getName());
        group.setType("organization");
        identityService.saveGroup(group);
        
        // 同步组织成员
        syncOrganizationMembers(org);
    }

    /**
     * 同步组织成员
     */
    private void syncOrganizationMembers(Organization org) {
        List<String> memberIds = organizationApi.getOrganizationMembers(org.getId());
        
        for (String memberId : memberIds) {
            boolean isMember = identityService.createGroupQuery()
                    .groupId(org.getId())
                    .groupMember(memberId)
                    .singleResult() != null;
            
            if (!isMember) {
                identityService.createMembership(memberId, org.getId());
            }
        }
    }
}
```

### 权限集成

#### 自定义权限处理器

```java
@Component
public class CustomPermissionHandler {

    @Autowired
    private PermissionApi permissionApi;

    /**
     * 检查用户是否有流程启动权限
     */
    public boolean canStartProcess(String userId, String processDefinitionKey) {
        return permissionApi.hasPermission(userId, "PROCESS_START", processDefinitionKey);
    }

    /**
     * 检查用户是否有任务办理权限
     */
    public boolean canCompleteTask(String userId, String taskId) {
        TaskService taskService = ProcessEngines.getDefaultProcessEngine().getTaskService();
        Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
        
        if (task == null) {
            return false;
        }
        
        // 检查是否是办理人
        if (userId.equals(task.getAssignee())) {
            return true;
        }
        
        // 检查是否是候选人
        List<IdentityLink> identityLinks = taskService.getIdentityLinksForTask(taskId);
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

    /**
     * 检查用户是否在用户组中
     */
    private boolean isUserInGroup(String userId, String groupId) {
        IdentityService identityService = ProcessEngines.getDefaultProcessEngine().getIdentityService();
        return identityService.createGroupQuery()
                .groupId(groupId)
                .groupMember(userId)
                .singleResult() != null;
    }

    /**
     * 检查用户是否有流程查看权限
     */
    public boolean canViewProcess(String userId, String processInstanceId) {
        RuntimeService runtimeService = ProcessEngines.getDefaultProcessEngine().getRuntimeService();
        ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
                .processInstanceId(processInstanceId)
                .singleResult();
        
        if (processInstance == null) {
            return false;
        }
        
        // 检查是否是发起人
        String startUserId = (String) runtimeService.getVariable(processInstanceId, "startUserId");
        if (userId.equals(startUserId)) {
            return true;
        }
        
        // 检查业务权限
        String businessKey = processInstance.getBusinessKey();
        return permissionApi.hasBusinessPermission(userId, businessKey);
    }
}
```

#### 流程启动权限拦截器

```java
@Component
public class ProcessStartInterceptor implements ExecutionListener {

    @Autowired
    private CustomPermissionHandler permissionHandler;

    @Override
    public void notify(DelegateExecution execution) {
        String userId = Authentication.getAuthenticatedUserId();
        String processDefinitionKey = execution.getProcessDefinitionId().split(":")[0];
        
        if (!permissionHandler.canStartProcess(userId, processDefinitionKey)) {
            throw new FlowableException("用户没有启动该流程的权限");
        }
    }
}
```
