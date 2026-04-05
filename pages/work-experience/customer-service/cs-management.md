\n\n    
    
    客服管理 - 客服系统
    
    
    
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
        
            
                客服管理
            
            CS Management - 客服人员管理与服务能力配置
            
            
                
                    
                    核心功能：支持第三方认证系统对接、客服服务能力评估、会话记录保存、技能标签管理、在线状态监控。
                
            

            
                功能概览
            
            
            
                
                    
                    第三方认证
                    多认证系统对接
                
                
                    
                    服务能力
                    客服能力评估
                
                
                    
                    会话记录
                    服务记录保存
                
                
                    
                    技能标签
                    技能标签管理
                
                
                    
                    在线状态
                    实时在线监控
                
                
                    
                    手动接入
                    手动接入排队用户
                
                
                    
                    第三方派单
                    外部系统派单
                
                
                    
                    数据统计
                    服务数据统计
                
            

            
                管理流程
            

            
                
                    
                    
                        
                            1
                            
                                第三方认证
                                对接企业认证系统，实现客服身份统一认证
                            
                        
                        
                            2
                            
                                服务能力配置
                                设置客服最大接待数、技能标签、在线状态
                            
                        
                        
                            3
                            
                                服务监控
                                实时监控客服在线状态、会话数量、服务质量
                            
                        
                        
                            4
                            
                                记录保存
                                所有客服服务记录保存到Redis，支持查询和统计
                            
                        
                    
                
            

            
                核心代码示例
            

            
                
                    
                        第三方认证服务
                        ThirdPartyAuthService.java
                    
                    
/**
 * 第三方认证服务 - 支持多认证系统对接
 */
@Service
@Slf4j
public class ThirdPartyAuthService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;
    
    private Map&lt;String, AuthProvider&gt; authProviders;

    public ThirdPartyAuthService() {
        authProviders = new HashMap&lt;&gt;();
        authProviders.put("ldap", new LdapAuthProvider());
        authProviders.put("oauth", new OAuthAuthProvider());
        authProviders.put("sso", new SsoAuthProvider());
    }

    public CsUser authenticate(String provider, String username, String password) throws Exception {
        AuthProvider authProvider = authProviders.get(provider);
        if (authProvider == null) {
            throw new IllegalArgumentException("不支持的认证方式: " + provider);
        }

        // 调用第三方认证
        AuthResult result = authProvider.authenticate(username, password);
        if (!result.isSuccess()) {
            throw new AuthenticationException("认证失败");
        }

        // 获取客服信息
        CsUser csUser = getCsUserInfo(result.getUserId());
        
        // 保存到Redis
        String csKey = "cs_info:" + csUser.getCsId();
        redisTemplate.opsForHash().putAll(csKey, BeanUtils.beanToMap(csUser));
        redisTemplate.expire(csKey, 24, TimeUnit.HOURS);

        // 生成会话token
        String sessionToken = UUID.randomUUID().toString();
        redisTemplate.opsForValue().set("cs_session:" + sessionToken, csUser.getCsId(), 12, TimeUnit.HOURS);
        
        csUser.setSessionToken(sessionToken);
        return csUser;
    }

    private CsUser getCsUserInfo(String userId) {
        // 从企业系统获取客服详细信息
        return CsUser.builder()
            .csId(userId)
            .name("客服" + userId)
            .capacity(5)
            .skills(Arrays.asList("通用", "技术"))
            .status(CsStatus.ONLINE)
            .build();
    }
}
                    
                

                
                    
                        客服服务能力管理
                        CsCapacityService.java
                    
                    
/**
 * 客服服务能力管理服务
 */
@Service
@Slf4j
public class CsCapacityService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;

    public void updateCapacity(String csId, int capacity) {
        String csKey = "cs_info:" + csId;
        redisTemplate.opsForHash().put(csKey, "capacity", capacity);
        log.info("客服 {} 服务能力更新为 {}", csId, capacity);
    }

    public void updateStatus(String csId, CsStatus status) {
        String csKey = "cs_info:" + csId;
        redisTemplate.opsForHash().put(csKey, "status", status.name());
        
        // 离线时自动将会话转接
        if (status == CsStatus.OFFLINE) {
            transferSessions(csId);
        }
    }

    public boolean hasAvailableCapacity(String csId) {
        String csKey = "cs_info:" + csId;
        Integer capacity = (Integer) redisTemplate.opsForHash().get(csKey, "capacity");
        Integer currentSessions = (Integer) redisTemplate.opsForHash().get(csKey, "current_sessions");
        
        return currentSessions != null &amp;&amp; capacity != null &amp;&amp; currentSessions &lt; capacity;
    }

    public List&lt;CsUser&gt; getAvailableCs(String skill) {
        // 获取所有在线且有剩余能力的客服
        Set&lt;String&gt; csKeys = redisTemplate.keys("cs_info:*");
        List&lt;CsUser&gt; availableCs = new ArrayList&lt;&gt;();
        
        for (String key : csKeys) {
            Map&lt;Object, Object&gt; csData = redisTemplate.opsForHash().entries(key);
            CsUser csUser = BeanUtils.mapToBean(csData, CsUser.class);
            
            if (csUser.getStatus() == CsStatus.ONLINE 
                &amp;&amp; hasAvailableCapacity(csUser.getCsId())
                &amp;&amp; csUser.getSkills().contains(skill)) {
                availableCs.add(csUser);
            }
        }
        
        // 按当前会话数排序，优先分配给会话少的客服
        availableCs.sort(Comparator.comparingInt(CsUser::getCurrentSessions));
        return availableCs;
    }

    public void incrementSession(String csId) {
        String csKey = "cs_info:" + csId;
        redisTemplate.opsForHash().increment(csKey, "current_sessions", 1);
    }

    public void decrementSession(String csId) {
        String csKey = "cs_info:" + csId;
        redisTemplate.opsForHash().increment(csKey, "current_sessions", -1);
    }
}
                    
                

                
                    
                        会话记录服务
                        SessionRecordService.java
                    
                    
/**
 * 会话记录服务 - 保存客服服务记录
 */
@Service
@Slf4j
public class SessionRecordService {

    @Autowired
    private RedisTemplate&lt;String, Object&gt; redisTemplate;

    public void createSessionRecord(SessionRecord record) {
        record.setSessionId(UUID.randomUUID().toString());
        record.setStartTime(LocalDateTime.now());
        
        // 保存到Redis Hash
        String recordKey = "session_record:" + record.getSessionId();
        redisTemplate.opsForHash().putAll(recordKey, BeanUtils.beanToMap(record));
        redisTemplate.expire(recordKey, 30, TimeUnit.DAYS);
        
        // 添加到客服会话列表
        String csSessionsKey = "cs_sessions:" + record.getCsId();
        redisTemplate.opsForList().rightPush(csSessionsKey, record.getSessionId());
        redisTemplate.expire(csSessionsKey, 30, TimeUnit.DAYS);
        
        log.info("会话记录创建: {}", record.getSessionId());
    }

    public void updateSessionRecord(String sessionId, Map&lt;String, Object&gt; updates) {
        String recordKey = "session_record:" + sessionId;
        redisTemplate.opsForHash().putAll(recordKey, updates);
    }

    public void endSession(String sessionId, String satisfaction) {
        String recordKey = "session_record:" + sessionId;
        Map&lt;String, Object&gt; updates = new HashMap&lt;&gt;();
        updates.put("endTime", LocalDateTime.now());
        updates.put("satisfaction", satisfaction);
        updates.put("status", "ENDED");
        
        updateSessionRecord(sessionId, updates);
        log.info("会话结束: {}", sessionId);
    }

    public List&lt;SessionRecord&gt; getCsSessionRecords(String csId, LocalDate date) {
        String csSessionsKey = "cs_sessions:" + csId;
        List&lt;String&gt; sessionIds = (List&lt;String&gt;) redisTemplate.opsForList().range(csSessionsKey, 0, -1);
        
        List&lt;SessionRecord&gt; records = new ArrayList&lt;&gt;();
        for (String sessionId : sessionIds) {
            String recordKey = "session_record:" + sessionId;
            Map&lt;Object, Object&gt; data = redisTemplate.opsForHash().entries(recordKey);
            SessionRecord record = BeanUtils.mapToBean(data, SessionRecord.class);
            
            // 按日期过滤
            if (record.getStartTime().toLocalDate().equals(date)) {
                records.add(record);
            }
        }
        
        return records;
    }

    public SessionStats getCsStats(String csId, LocalDate date) {
        List&lt;SessionRecord&gt; records = getCsSessionRecords(csId, date);
        
        long totalSessions = records.size();
        long avgDuration = records.stream()
            .filter(r -&gt; r.getEndTime() != null)
            .mapToLong(r -&gt; Duration.between(r.getStartTime(), r.getEndTime()).getSeconds())
            .average()
            .orElse(0);
        double avgSatisfaction = records.stream()
            .filter(r -&gt; r.getSatisfaction() != null)
            .mapToInt(r -&gt; Integer.parseInt(r.getSatisfaction()))
            .average()
            .orElse(0);
        
        return SessionStats.builder()
            .csId(csId)
            .date(date)
            .totalSessions(totalSessions)
            .avgDuration(avgDuration)
            .avgSatisfaction(avgSatisfaction)
            .build();
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    第三方认证
                    多认证系统灵活对接
                
                
                    
                    能力评估
                    智能服务能力管理
                
                
                    
                    记录保存
                    全量会话记录持久化
                
                
                    
                    技能匹配
                    技能标签智能匹配
                
                
                    
                    第三方派单
                    外部系统派单支持
                
                
                    
                    数据统计
                    服务数据全面统计
