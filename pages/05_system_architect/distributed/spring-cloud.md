Spring Cloud - 架构师学习笔记
    
    


    
        
            
                Spring Cloud
            
            
            
                
                    Spring Cloud为开发者提供了在分布式系统（如配置管理、服务发现、断路器、智能路由、微代理、控制总线、一次性令牌、全局锁、领导选举、分布式会话、集群状态）操作的开发工具。使用Spring Boot开发风格，使分布式系统中的协调变得更容易。
                
                
                Spring Cloud核心组件
                
                
                    
                        
                            
                                组件
                                功能
                                实现方案
                            
                        
                        
                            
                                服务注册与发现
                                服务实例的自动注册和发现
                                Eureka, Consul, Nacos
                            
                            
                                客户端负载均衡
                                在客户端实现负载均衡算法
                                Ribbon, Spring Cloud LoadBalancer
                            
                            
                                服务调用
                                简化服务间的调用
                                OpenFeign
                            
                            
                                熔断器
                                提供服务降级和熔断机制
                                Hystrix, Resilience4j
                            
                            
                                API网关
                                统一服务入口，提供路由、过滤等功能
                                Zuul, Spring Cloud Gateway
                            
                            
                                配置管理
                                集中管理分布式系统的配置
                                Spring Cloud Config, Nacos
                            
                            
                                消息总线
                                用于传播集群状态变化
                                Spring Cloud Bus
                            
                            
                                安全控制
                                提供安全控制和认证授权
                                Spring Cloud Security
                            
                        
                    
                
                
                服务注册与发现
                
                
                    
                        
                            
                            Eureka
                        
                        
                            Netflix开源的服务发现组件，是Spring Cloud Netflix的核心组件之一。
                        
                        
// Eureka Server配置
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// Eureka Client配置
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
                    
                    
                    
                        
                            
                            Nacos
                        
                        
                            Alibaba开源的动态服务发现、配置管理和服务管理平台。
                        
                        
// Nacos配置
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
                    
                
                
                服务调用
                
                
                    
                        
                        OpenFeign
                    
                    
                        声明式的Web服务客户端，使得编写Web服务客户端变得更加简单。
                    
                    
                    
// 定义Feign客户端
@FeignClient(name = "user-service")
public interface UserServiceClient {
    
    @GetMapping("/users/{id}")
    User getUserById(@PathVariable("id") Long id);
    
    @PostMapping("/users")
    User createUser(@RequestBody User user);
}

// 使用Feign客户端
@RestController
public class OrderController {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable("id") Long id) {
        // 调用用户服务
        User user = userServiceClient.getUserById(1L);
        // 处理订单逻辑
        return orderService.getOrder(id);
    }
}
                
                
                熔断器
                
                
                    
                        
                            
                            Hystrix
                        
                        
                            Netflix开源的容错库，通过添加延迟容忍和容错逻辑来帮助控制分布式服务之间的交互。
                        
                        
// 使用HystrixCommand注解
@RestController
public class UserController {
    
    @HystrixCommand(fallbackMethod = "getUserFallback")
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable("id") Long id) {
        // 调用用户服务
        return userService.getUserById(id);
    }
    
    // 降级方法
    public User getUserFallback(Long id) {
        return new User(id, "默认用户", "default@example.com");
    }
}
                    
                    
                    
                        
                            
                            Resilience4j
                        
                        
                            轻量级的容错库，专为Java 8和函数式编程设计。
                        
                        
// 使用Resilience4j注解
@RestController
public class ProductController {
    
    @Retry(name = "product-service")
    @CircuitBreaker(name = "product-service", fallbackMethod = "getProductFallback")
    @GetMapping("/products/{id}")
    public Product getProduct(@PathVariable("id") Long id) {
        return productService.getProductById(id);
    }
    
    public Product getProductFallback(Long id, Exception ex) {
        return new Product(id, "默认产品", 0.0);
    }
}
                    
                
                
                API网关
                
                
                    
                        
                        Spring Cloud Gateway
                    
                    
                        基于Spring Framework 5、Project Reactor和Spring Boot 2.0构建的API网关。
                    
                    
                    
// Gateway配置
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r.path("/users/**")
                .uri("lb://user-service"))
            .route("order-service", r -> r.path("/orders/**")
                .uri("lb://order-service"))
            .build();
    }
}

// application.yml配置
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/users/**
          filters:
            - StripPrefix=1
                
                
                配置管理
                
                
                    
                        
                        Spring Cloud Config
                    
                    
                        为分布式系统中的外部化配置提供服务器端和客户端支持。
                    
                    
                    
// Config Server配置
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// Config Client配置
@RestController
@RefreshScope
public class ConfigController {
    
    @Value("${config.info}")
    private String configInfo;
    
    @GetMapping("/config")
    public String getConfigInfo() {
        return configInfo;
    }
}

// bootstrap.yml配置
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://localhost:8888
      profile: dev
                
                
                Spring Cloud最佳实践
                
                    
                        合理选择Spring Cloud版本，注意组件间的兼容性
                        使用Spring Cloud Alibaba替代部分Netflix组件，提高稳定性
                        实施健康检查和心跳机制，确保服务可用性
                        配置合理的超时时间和重试机制
                        建立完善的监控体系，使用Spring Boot Admin等工具
                        实施蓝绿部署和金丝雀发布，降低发布风险
                        使用分布式链路追踪工具（如Sleuth + Zipkin）定位问题
