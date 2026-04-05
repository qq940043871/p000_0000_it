\n\n    
    
    中间件网关 - B2B分布式项目
    
    
    
        .filter-step {
            transition: all 0.3s ease;
        }
        .filter-step:hover {
            transform: translateY(-5px);
        }
    \n\n    
        
            
                中间件网关
            
            Middleware Gateway - 统一服务入口与流量治理
            
            
                
                    
                    核心定位：中间件网关作为系统对外的统一入口，提供HTTP接口、认证校验、权限拦截、IP安全、敏感信息过滤、协议授权、登录级别校验、协议权限级别校验和缓存功能，保障系统安全、稳定、高效运行。
                
            

            
                请求处理流程
            
            
            
                
                    
                        
                        请求入口
                    
                    
                        
                        认证校验
                    
                    
                        
                        权限拦截
                    
                    
                        
                        IP拦截
                    
                    
                        
                        敏感过滤
                    
                    
                        
                        协议授权
                    
                    
                        
                        缓存检查
                    
                    
                        
                        转发服务
                    
                
            

            
                核心功能模块
            
            
            
                
                    
                        
                            
                        
                        HTTP接口
                    
                    
                        对外提供统一的HTTP接口，支持RESTful API，接口文档自动生成。
                    
                    
                        @RestController
@RequestMapping("/api/gateway")
public class GatewayController {
    
    @Autowired
    private GatewayService gatewayService;
    
    @PostMapping("/{service}/{method}")
    public ResponseEntity&lt;?&gt; invoke(
            @PathVariable String service,
            @PathVariable String method,
            @RequestBody Map&lt;String, Object&gt; params,
            HttpServletRequest request) {
        
        GatewayRequest gatewayRequest = new GatewayRequest();
        gatewayRequest.setService(service);
        gatewayRequest.setMethod(method);
        gatewayRequest.setParams(params);
        gatewayRequest.setRequestId(UUID.randomUUID().toString());
        gatewayRequest.setTimestamp(System.currentTimeMillis());
        
        GatewayResponse response = gatewayService.invoke(gatewayRequest, request);
        return ResponseEntity.ok(response);
    }
}
                    
                
                
                
                    
                        
                            
                        
                        认证校验
                    
                    
                        支持JWT、OAuth2.0等多种认证方式，验证请求的合法性。
                    
                    
                        @Component
public class AuthFilter implements GlobalFilter {
    
    @Autowired
    private JwtService jwtService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        
        if (StringUtils.isEmpty(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        try {
            Claims claims = jwtService.verifyToken(token);
            exchange.getAttributes().put("userId", claims.getSubject());
            exchange.getAttributes().put("roles", claims.get("roles"));
        } catch (Exception e) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
}
                    
                
                
                
                    
                        
                            
                        
                        权限拦截
                    
                    
                        基于RBAC模型的权限控制，检查用户是否拥有访问接口的权限。
                    
                    
                        @Component
public class PermissionFilter implements GlobalFilter {
    
    @Autowired
    private PermissionService permissionService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String userId = exchange.getAttribute("userId");
        String path = exchange.getRequest().getPath().value();
        String method = exchange.getRequest().getMethod().name();
        
        boolean hasPermission = permissionService.checkPermission(userId, path, method);
        
        if (!hasPermission) {
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
}
                    
                
                
                
                    
                        
                            
                        
                        IP安全拦截
                    
                    
                        IP白名单/黑名单机制，防止恶意IP访问，保障系统安全。
                    
                    
                        @Component
public class IpFilter implements GlobalFilter {
    
    @Autowired
    private IpService ipService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String clientIp = getClientIp(exchange.getRequest());
        
        if (ipService.isBlacklisted(clientIp)) {
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            return exchange.getResponse().setComplete();
        }
        
        if (ipService.requiresWhitelist(exchange.getRequest().getPath().value())) {
            if (!ipService.isWhitelisted(clientIp)) {
                exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
                return exchange.getResponse().setComplete();
            }
        }
        
        return chain.filter(exchange);
    }
    
    private String getClientIp(ServerHttpRequest request) {
        String xForwardedFor = request.getHeaders().getFirst("X-Forwarded-For");
        if (StringUtils.isNotEmpty(xForwardedFor)) {
            return xForwardedFor.split(",")[0].trim();
        }
        return request.getRemoteAddress().getAddress().getHostAddress();
    }
}
                    
                
                
                
                    
                        
                            
                        
                        敏感信息过滤
                    
                    
                        对请求和响应中的敏感信息进行初步过滤和脱敏处理。
                    
                    
                        @Component
public class SensitiveFilter implements GlobalFilter {
    
    @Autowired
    private SensitiveWordService sensitiveWordService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        if (request.getMethod() == HttpMethod.POST || request.getMethod() == HttpMethod.PUT) {
            return DataBufferUtils.join(request.getBody())
                .flatMap(dataBuffer -&gt; {
                    byte[] bytes = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(bytes);
                    DataBufferUtils.release(dataBuffer);
                    
                    String body = new String(bytes, StandardCharsets.UTF_8);
                    String filteredBody = sensitiveWordService.filter(body);
                    
                    ServerHttpRequest mutatedRequest = request.mutate()
                        .body(BodyInserters.fromValue(filteredBody))
                        .build();
                    
                    return chain.filter(exchange.mutate().request(mutatedRequest).build());
                });
        }
        
        return chain.filter(exchange);
    }
}

@Service
public class SensitiveWordService {
    
    private Set&lt;String&gt; sensitiveWords = new HashSet&lt;&gt;(Arrays.asList(
        "密码", "身份证号", "银行卡号", "手机号"
    ));
    
    public String filter(String content) {
        for (String word : sensitiveWords) {
            content = content.replaceAll(word, "***");
        }
        return content;
    }
}
                    
                
                
                
                    
                        
                            
                        
                        协议授权
                    
                    
                        基于协议的授权机制，控制不同协议的访问权限。
                    
                    
                        @Component
public class ProtocolAuthFilter implements GlobalFilter {
    
    @Autowired
    private ProtocolAuthService protocolAuthService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String protocol = exchange.getRequest().getHeaders().getFirst("X-Protocol");
        String appId = exchange.getRequest().getHeaders().getFirst("X-App-Id");
        
        if (StringUtils.isNotEmpty(protocol)) {
            boolean authorized = protocolAuthService.checkProtocolAuth(appId, protocol);
            
            if (!authorized) {
                exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
                return exchange.getResponse().setComplete();
            }
        }
        
        return chain.filter(exchange);
    }
}

@Service
public class ProtocolAuthService {
    
    @Autowired
    private ProtocolAuthRepository protocolAuthRepository;
    
    public boolean checkProtocolAuth(String appId, String protocol) {
        ProtocolAuth auth = protocolAuthRepository.findByAppIdAndProtocol(appId, protocol);
        return auth != null &amp;&amp; auth.isEnabled();
    }
}
                    
                
                
                
                    
                        
                            
                        
                        登录级别校验
                    
                    
                        根据用户登录级别（如普通登录、强认证登录）控制接口访问权限。
                    
                    
                        @Component
public class LoginLevelFilter implements GlobalFilter {
    
    @Autowired
    private LoginLevelService loginLevelService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String userId = exchange.getAttribute("userId");
        String path = exchange.getRequest().getPath().value();
        
        LoginLevel requiredLevel = loginLevelService.getRequiredLevel(path);
        LoginLevel userLevel = loginLevelService.getUserLevel(userId);
        
        if (userLevel.getLevel() &lt; requiredLevel.getLevel()) {
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
}

public enum LoginLevel {
    NORMAL(1, "普通登录"),
    STRONG(2, "强认证登录"),
    MFA(3, "多因素认证");
    
    private final int level;
    private final String desc;
    
    LoginLevel(int level, String desc) {
        this.level = level;
        this.desc = desc;
    }
    
    public int getLevel() {
        return level;
    }
}
                    
                
                
                
                    
                        
                            
                        
                        协议权限级别校验
                    
                    
                        基于协议权限级别的精细控制，支持多级权限体系。
                    
                    
                        @Component
public class ProtocolPermissionFilter implements GlobalFilter {
    
    @Autowired
    private ProtocolPermissionService protocolPermissionService;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String userId = exchange.getAttribute("userId");
        String protocol = exchange.getRequest().getHeaders().getFirst("X-Protocol");
        String action = exchange.getRequest().getHeaders().getFirst("X-Action");
        
        ProtocolPermissionLevel requiredLevel = 
            protocolPermissionService.getRequiredLevel(protocol, action);
        ProtocolPermissionLevel userLevel = 
            protocolPermissionService.getUserLevel(userId, protocol);
        
        if (userLevel.getLevel() &lt; requiredLevel.getLevel()) {
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }
}

public enum ProtocolPermissionLevel {
    READ(1, "只读"),
    WRITE(2, "读写"),
    ADMIN(3, "管理"),
    SUPER(4, "超级");
    
    private final int level;
    private final String desc;
    
    ProtocolPermissionLevel(int level, String desc) {
        this.level = level;
        this.desc = desc;
    }
    
    public int getLevel() {
        return level;
    }
}
                    
                
                
                
                    
                        
                            
                        
                        协议缓存功能
                    
                    
                        对部分协议的响应进行缓存，提升系统性能，减少后端服务压力。
                    
                    
                        @Component
public class CacheFilter implements GlobalFilter {
    
    @Autowired
    private CacheService cacheService;
    
    @Autowired
    private RouteLocator routeLocator;
    
    @Override
    public Mono&lt;Void&gt; filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        if (request.getMethod() != HttpMethod.GET) {
            return chain.filter(exchange);
        }
        
        String cacheKey = generateCacheKey(request);
        CacheConfig cacheConfig = cacheService.getCacheConfig(request.getPath().value());
        
        if (cacheConfig == null || !cacheConfig.isEnabled()) {
            return chain.filter(exchange);
        }
        
        String cachedResponse = cacheService.get(cacheKey);
        if (StringUtils.isNotEmpty(cachedResponse)) {
            exchange.getResponse().getHeaders().add("X-Cache", "HIT");
            exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);
            return exchange.getResponse().writeWith(
                Mono.just(exchange.getResponse().bufferFactory().wrap(
                    cachedResponse.getBytes(StandardCharsets.UTF_8)
                ))
            );
        }
        
        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(exchange.getResponse()) {
            @Override
            public Mono&lt;Void&gt; writeWith(Publisher&lt;? extends DataBuffer&gt; body) {
                if (body instanceof Flux) {
                    Flux&lt;? extends DataBuffer&gt; fluxBody = (Flux&lt;? extends DataBuffer&gt;) body;
                    return super.writeWith(fluxBody.buffer().map(dataBuffers -&gt; {
                        DataBufferFactory dataBufferFactory = exchange.getResponse().bufferFactory();
                        DataBuffer joined = dataBufferFactory.join(dataBuffers);
                        byte[] content = new byte[joined.readableByteCount()];
                        joined.read(content);
                        DataBufferUtils.release(joined);
                        
                        String responseBody = new String(content, StandardCharsets.UTF_8);
                        cacheService.put(cacheKey, responseBody, cacheConfig.getTtl());
                        
                        exchange.getResponse().getHeaders().add("X-Cache", "MISS");
                        return dataBufferFactory.wrap(content);
                    }));
                }
                return super.writeWith(body);
            }
        };
        
        return chain.filter(exchange.mutate().response(decoratedResponse).build());
    }
    
    private String generateCacheKey(ServerHttpRequest request) {
        StringBuilder key = new StringBuilder();
        key.append(request.getMethod()).append(":");
        key.append(request.getPath()).append(":");
        key.append(request.getQueryParams().toSingleValueMap().toString());
        return DigestUtils.md5DigestAsHex(key.toString().getBytes());
    }
}

@Service
public class CacheService {
    
    @Autowired
    private RedisTemplate&lt;String, String&gt; redisTemplate;
    
    private Map&lt;String, CacheConfig&gt; cacheConfigs = new HashMap&lt;&gt;();
    
    @PostConstruct
    public void init() {
        cacheConfigs.put("/api/gateway/dict/list", 
            new CacheConfig(true, 300, "字典列表"));
        cacheConfigs.put("/api/gateway/config/get", 
            new CacheConfig(true, 600, "配置获取"));
    }
    
    public CacheConfig getCacheConfig(String path) {
        return cacheConfigs.get(path);
    }
    
    public String get(String key) {
        return redisTemplate.opsForValue().get("cache:" + key);
    }
    
    public void put(String key, String value, long ttl) {
        redisTemplate.opsForValue().set("cache:" + key, value, ttl, TimeUnit.SECONDS);
    }
}

public class CacheConfig {
    private boolean enabled;
    private long ttl;
    private String description;
    
    public CacheConfig(boolean enabled, long ttl, String description) {
        this.enabled = enabled;
        this.ttl = ttl;
        this.description = description;
    }
    
    public boolean isEnabled() {
        return enabled;
    }
    
    public long getTtl() {
        return ttl;
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    安全防护
                    多层安全防护，保障系统安全
                
                
                    
                    性能优化
                    缓存机制减少后端压力
                
                
                    
                    灵活配置
                    动态调整安全策略和缓存配置
                
                
                    
                    可观测性
                    完整的请求日志和监控指标
                
                
                    
                    敏感过滤
                    数据脱敏，保护用户隐私
                
                
                    
                    权限分级
                    多级权限控制，精细管理
                
            

            
                技术实现
            
            
            
                
                    
                        网关框架
                    
                    
                        
                            Spring Cloud Gateway
                            推荐
                        
                        
                            Zuul
                            可选
                        
                        
                            Kong
                            高性能
                        
                    
                
                
                
                    
                        核心组件
                    
                    
                        
                            GlobalFilter
                            全局过滤器
                        
                        
                            GatewayFilterChain
                            过滤链
                        
                        
                            RouteLocator
                            路由定位
