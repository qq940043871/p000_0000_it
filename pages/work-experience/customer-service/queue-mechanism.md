\n\n    
    
    排队机制 - 客服系统
    
    
    
        .feature-card {
            transition: all 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
        .code-block {
            background: #1e293b;
            color: #e2e8f0;
            font-family: 'Consolas', 'Monaco', monospace;
            font-size: 13px;
            line-height: 1.6;
            overflow-x: auto;
        }
    \n\n    
        
            
                排队机制
            
            Queue Mechanism - 智能排队与分配策略
            
            
                
                    
                    核心功能：支持智能排队分配、手动接入排队用户、第三方系统派单、优先级队列、超时提醒、实时队列位置通知。
                
            

            
                功能概览
            
            
            
                
                    
                    智能排队
                    FIFO优先级队列
                
                
                    
                    手动接入
                    手动接入排队用户
                
                
                    
                    第三方派单
                    外部系统派单
                
                
                    
                    优先级
                    VIP用户优先
                
                
                    
                    超时提醒
                    排队超时提醒
                
                
                    
                    位置通知
                    实时队列位置
                
                
                    
                    技能匹配
                    按技能分配
                
                
                    
                    负载均衡
                    客服负载均衡
                
            

            
                排队流程
            

            
                
                    
                    
                        
                            1
                            
                                用户进入队列
                                用户发起咨询请求，根据技能标签进入对应队列
                            
                        
                        
                            2
                            
                                智能分配
                                系统根据客服服务能力和技能匹配，优先分配给空闲客服
                            
                        
                        
                            3
                            
                                排队等待
                                无空闲客服时，用户进入排队队列，实时通知队列位置
                            
                        
                        
                            4
                            
                                手动接入/派单
                                客服可手动接入排队用户，或接收第三方系统派单
                            
                        
                        
                            5
                            
                                开始服务
                                用户与客服建立会话，开始服务
                            
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        智能排队服务
                        QueueService.java
                    
                    
/**
 * 智能排队服务 - 处理用户排队和分配
 */
@Service
@Slf4j
public class QueueService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;
    
    @Autowired
    private CsCapacityService csCapacityService;

    public String addToQueue(String userId, String skill, int priority) {
        String queueKey = "queue:" + skill;
        String queueId = UUID.randomUUID().toString();
        
        // 加入ZSet队列，score为优先级
        double score = calculateScore(priority, System.currentTimeMillis());
        redisTemplate.opsForZSet().add(queueKey, queueId, score);
        
        // 保存排队用户信息
        String userKey = "queue_user:" + queueId;
        Map&lt;String, Object&gt; userInfo = new HashMap&lt;&gt;();
        userInfo.put("userId", userId);
        userInfo.put("skill", skill);
        userInfo.put("joinTime", System.currentTimeMillis());
        userInfo.put("priority", priority);
        redisTemplate.opsForHash().putAll(userKey, userInfo);
        redisTemplate.expire(userKey, 24, TimeUnit.HOURS);
        
        // 尝试自动分配
        tryAssignFromQueue(skill);
        
        log.info("用户 {} 加入队列 {}", userId, skill);
        return queueId;
    }

    public void tryAssignFromQueue(String skill) {
        String queueKey = "queue:" + skill;
        
        // 获取可用客服
        List&lt;CsUser&gt; availableCs = csCapacityService.getAvailableCs(skill);
        if (availableCs.isEmpty()) {
            log.info("无可用客服，用户继续排队");
            return;
        }
        
        // 从队列头部获取用户
        Set&lt;String&gt; queueIds = redisTemplate.opsForZSet().range(queueKey, 0, 0);
        if (queueIds == null || queueIds.isEmpty()) {
            return;
        }
        
        String queueId = queueIds.iterator().next();
        CsUser csUser = availableCs.get(0);
        
        // 分配用户给客服
        assignUserToCs(queueId, csUser.getCsId());
        
        // 从队列移除
        redisTemplate.opsForZSet().remove(queueKey, queueId);
    }

    public void manualAssign(String queueId, String csId) {
        String queueKey = "queue:" + getQueueSkill(queueId);
        
        // 检查客服是否有可用能力
        if (!csCapacityService.hasAvailableCapacity(csId)) {
            throw new IllegalStateException("客服服务能力已满");
        }
        
        // 分配用户给客服
        assignUserToCs(queueId, csId);
        
        // 从队列移除
        redisTemplate.opsForZSet().remove(queueKey, queueId);
        
        log.info("手动接入: 队列ID {} -&gt; 客服ID {}", queueId, csId);
    }

    public void thirdPartyDispatch(DispatchRequest request) {
        String userId = request.getUserId();
        String skill = request.getSkill();
        String csId = request.getCsId();
        
        // 验证派单请求
        if (!csCapacityService.hasAvailableCapacity(csId)) {
            throw new IllegalStateException("客服服务能力已满");
        }
        
        // 直接分配
        String sessionId = createSession(userId, csId, skill);
        
        log.info("第三方派单: 用户ID {} -&gt; 客服ID {}", userId, csId);
    }

    public QueuePosition getQueuePosition(String queueId) {
        String skill = getQueueSkill(queueId);
        String queueKey = "queue:" + skill;
        
        // 获取当前位置
        Long rank = redisTemplate.opsForZSet().rank(queueKey, queueId);
        Long total = redisTemplate.opsForZSet().zCard(queueKey);
        
        return QueuePosition.builder()
            .queueId(queueId)
            .position(rank != null ? rank + 1 : 0)
            .total(total != null ? total : 0)
            .build();
    }

    private double calculateScore(int priority, long timestamp) {
        // 优先级越高，分数越低（ZSet按分数升序排列）
        // VIP用户优先级为10，普通用户为1
        return (100 - priority) * 1000000000000L + timestamp;
    }

    private String getQueueSkill(String queueId) {
        String userKey = "queue_user:" + queueId;
        return (String) redisTemplate.opsForHash().get(userKey, "skill");
    }

    private String assignUserToCs(String queueId, String csId) {
        String userKey = "queue_user:" + queueId;
        String userId = (String) redisTemplate.opsForHash().get(userKey, "userId");
        String skill = (String) redisTemplate.opsForHash().get(userKey, "skill");
        
        // 创建会话
        String sessionId = createSession(userId, csId, skill);
        
        // 增加客服会话数
        csCapacityService.incrementSession(csId);
        
        return sessionId;
    }

    private String createSession(String userId, String csId, String skill) {
        String sessionId = UUID.randomUUID().toString();
        
        // 保存会话信息
        String sessionKey = "session:" + sessionId;
        Map&lt;String, Object&gt; sessionInfo = new HashMap&lt;&gt;();
        sessionInfo.put("sessionId", sessionId);
        sessionInfo.put("userId", userId);
        sessionInfo.put("csId", csId);
        sessionInfo.put("skill", skill);
        sessionInfo.put("startTime", System.currentTimeMillis());
        sessionInfo.put("status", "ACTIVE");
        redisTemplate.opsForHash().putAll(sessionKey, sessionInfo);
        redisTemplate.expire(sessionKey, 24, TimeUnit.HOURS);
        
        return sessionId;
    }
}
                    
                

                
                    
                        超时监控服务
                        QueueTimeoutService.java
                    
                    
/**
 * 超时监控服务 - 处理排队超时提醒
 */
@Service
@Slf4j
public class QueueTimeoutService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;
    
    @Autowired
    private WebSocketServer webSocketServer;
    
    private static final long TIMEOUT_THRESHOLD = 5 * 60 * 1000;

    @Scheduled(fixedRate = 30000)
    public void checkTimeout() {
        log.info("检查排队超时...");
        
        // 遍历所有技能队列
        Set&lt;String&gt; queueKeys = redisTemplate.keys("queue:*");
        
        for (String queueKey : queueKeys) {
            Set&lt;String&gt; queueIds = redisTemplate.opsForZSet().range(queueKey, 0, -1);
            
            if (queueIds == null) {
                continue;
            }
            
            for (String queueId : queueIds) {
                checkUserTimeout(queueId);
            }
        }
    }

    private void checkUserTimeout(String queueId) {
        String userKey = "queue_user:" + queueId;
        Long joinTime = (Long) redisTemplate.opsForHash().get(userKey, "joinTime");
        
        if (joinTime == null) {
            return;
        }
        
        long waitTime = System.currentTimeMillis() - joinTime;
        
        // 超时提醒
        if (waitTime &gt; TIMEOUT_THRESHOLD) {
            String userId = (String) redisTemplate.opsForHash().get(userKey, "userId");
            
            // 发送超时提醒
            sendTimeoutNotification(userId, waitTime);
            
            log.warn("用户 {} 排队超时: {}ms", userId, waitTime);
        }
    }

    private void sendTimeoutNotification(String userId, long waitTime) {
        String message = String.format(
            "您已等待 %d 分钟，目前排队人数较多，请耐心等待...",
            waitTime / 60000
        );
        
        // 通过WebSocket发送提醒
        webSocketServer.sendMessage(userId, message);
    }
}
                    
                

                
                    
                        第三方派单接口
                        ThirdPartyDispatchController.java
                    
                    
/**
 * 第三方派单接口 - 支持外部系统派单
 */
@RestController
@RequestMapping("/api/third-party")
@Slf4j
public class ThirdPartyDispatchController {

    @Autowired
    private QueueService queueService;

    @PostMapping("/dispatch")
    public ResponseEntity&lt;?&gt; dispatch(
            @RequestBody DispatchRequest request,
            @RequestHeader("X-API-Key") String apiKey) {
        
        // 验证API Key
        if (!validateApiKey(apiKey)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
        
        // 执行派单
        try {
            queueService.thirdPartyDispatch(request);
            return ResponseEntity.ok().build();
        } catch (Exception e) {
            log.error("派单失败: {}", e.getMessage(), e);
            return ResponseEntity.badRequest().body(e.getMessage());
        }
    }

    @GetMapping("/queue-status")
    public ResponseEntity&lt;?&gt; getQueueStatus(
            @RequestParam String skill,
            @RequestHeader("X-API-Key") String apiKey) {
        
        // 验证API Key
        if (!validateApiKey(apiKey)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
        
        // 获取队列状态
        QueueStatus status = queueService.getQueueStatus(skill);
        return ResponseEntity.ok(status);
    }

    private boolean validateApiKey(String apiKey) {
        // 验证第三方API Key
        // 实际项目中从配置或数据库读取
        return "valid-api-key".equals(apiKey);
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    智能排队
                    FIFO+优先级队列
                
                
                    
                    手动接入
                    灵活手动接入
                
                
                    
                    第三方派单
                    外部系统对接
                
                
                    
                    优先级
                    VIP用户优先
                
                
                    
                    超时提醒
                    智能超时处理
                
                
                    
                    位置通知
                    实时队列位置
