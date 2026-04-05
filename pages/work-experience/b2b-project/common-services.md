\n\n    
    
    公共服务模块 - B2B分布式项目
    
    
    
        .service-card {
            transition: all 0.3s ease;
        }
        .service-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
    \n\n    
        
            
                公共服务模块
            
            Common Services - 通用基础服务能力
            
            
                
                    
                    核心定位：公共服务模块提供系统通用的基础服务能力，包括认证服务、短信服务、外联网关、支付模块、订单模块、客户模块、智能体模块等，减少重复开发，提升开发效率。
                
            

            
                服务模块概览
            
            
            
                
                    
                    认证服务
                    统一身份认证
                
                
                    
                    短信服务
                    短信发送与验证码
                
                
                    
                    外联网关
                    第三方系统对接
                
                
                    
                    支付模块
                    多渠道支付
                
                
                    
                    订单模块
                    订单全生命周期
                
                
                    
                    客户模块
                    客户关系管理
                
                
                    
                    智能体模块
                    AI智能助手
                
                
                    
                    业务模块
                    各业务领域服务
                
            

            
                核心服务模块
            
            
            
                
                    
                        
                            
                        
                        认证服务
                    
                    
                        提供统一的身份认证、授权管理、Token管理等功能，支持多种认证方式。
                    
                    
                        @Service
public class AuthService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private JwtService jwtService;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    public LoginResponse login(LoginRequest request) {
        User user = userRepository.findByUsername(request.getUsername());
        
        if (user == null || !passwordEncoder.matches(request.getPassword(), user.getPassword())) {
            throw new BusinessException("用户名或密码错误");
        }
        
        if (!user.isEnabled()) {
            throw new BusinessException("账号已禁用");
        }
        
        String accessToken = jwtService.generateAccessToken(user);
        String refreshToken = jwtService.generateRefreshToken(user);
        
        return LoginResponse.builder()
            .accessToken(accessToken)
            .refreshToken(refreshToken)
            .userInfo(convertToUserInfo(user))
            .build();
    }
    
    public RefreshTokenResponse refreshToken(String refreshToken) {
        if (!jwtService.validateRefreshToken(refreshToken)) {
            throw new BusinessException("RefreshToken无效");
        }
        
        String userId = jwtService.getUserIdFromToken(refreshToken);
        User user = userRepository.findById(userId).orElse(null);
        
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        
        String newAccessToken = jwtService.generateAccessToken(user);
        
        return RefreshTokenResponse.builder()
            .accessToken(newAccessToken)
            .build();
    }
    
    public void logout(String userId) {
        jwtService.revokeTokens(userId);
    }
}
                    
                

                
                    
                        
                            
                        
                        短信服务
                    
                    
                        提供短信发送、验证码管理、模板管理等功能，支持多渠道短信服务商接入。
                    
                    
                        @Service
public class SmsService {
    
    @Autowired
    private SmsProvider smsProvider;
    
    @Autowired
    private RedisTemplate redisTemplate;
    
    private static final String SMS_CODE_PREFIX = "sms:code:";
    
    public void sendVerificationCode(String phone, String scene) {
        String code = generateCode(6);
        
        String templateCode = getTemplateCode(scene);
        Map params = new HashMap();
        params.put("code", code);
        
        smsProvider.send(phone, templateCode, params);
        
        String redisKey = SMS_CODE_PREFIX + scene + ":" + phone;
        redisTemplate.opsForValue().set(redisKey, code, 5, TimeUnit.MINUTES);
    }
    
    public boolean verifyCode(String phone, String code, String scene) {
        String redisKey = SMS_CODE_PREFIX + scene + ":" + phone;
        String storedCode = redisTemplate.opsForValue().get(redisKey);
        
        if (storedCode == null) {
            return false;
        }
        
        boolean matched = storedCode.equals(code);
        if (matched) {
            redisTemplate.delete(redisKey);
        }
        
        return matched;
    }
    
    public void sendNotification(String phone, String templateCode, Map params) {
        smsProvider.send(phone, templateCode, params);
    }
    
    private String generateCode(int length) {
        StringBuilder sb = new StringBuilder();
        Random random = new Random();
        for (int i = 0; i  templateMap = new HashMap();
        templateMap.put("login", "TPL_LOGIN");
        templateMap.put("register", "TPL_REGISTER");
        templateMap.put("reset", "TPL_RESET");
        return templateMap.getOrDefault(scene, "TPL_DEFAULT");
    }
}

public interface SmsProvider {
    void send(String phone, String templateCode, Map params);
}

@Service
public class AliyunSmsProvider implements SmsProvider {
    
    @Override
    public void send(String phone, String templateCode, Map params) {
    }
}
                    
                

                
                    
                        
                            
                        
                        外联网关
                    
                    
                        统一对接各种第三方系统，提供适配器模式，支持协议转换、签名验签、限流熔断等功能。
                    
                    
                        @Service
public class ExternalGateway {
    
    private final Map adapters = new ConcurrentHashMap();
    
    @PostConstruct
    public void init() {
        adapters.put("alipay", new AlipayAdapter());
        adapters.put("wechat", new WechatAdapter());
        adapters.put("sms", new SmsAdapter());
        adapters.put("logistics", new LogisticsAdapter());
    }
    
    public ExternalResponse invoke(String system, String action, Map params) {
        ExternalAdapter adapter = adapters.get(system);
        
        if (adapter == null) {
            throw new BusinessException("不支持的第三方系统: " + system);
        }
        
        try {
            return adapter.execute(action, params);
        } catch (Exception e) {
            return ExternalResponse.error(e.getMessage());
        }
    }
    
    public void registerAdapter(String system, ExternalAdapter adapter) {
        adapters.put(system, adapter);
    }
}

public interface ExternalAdapter {
    ExternalResponse execute(String action, Map params);
    String getSystemName();
}

@Component
public class AlipayAdapter implements ExternalAdapter {
    
    @Override
    public ExternalResponse execute(String action, Map params) {
        switch (action) {
            case "pay":
                return pay(params);
            case "query":
                return query(params);
            case "refund":
                return refund(params);
            default:
                return ExternalResponse.error("不支持的操作: " + action);
        }
    }
    
    private ExternalResponse pay(Map params) {
        return ExternalResponse.success();
    }
    
    private ExternalResponse query(Map params) {
        return ExternalResponse.success();
    }
    
    private ExternalResponse refund(Map params) {
        return ExternalResponse.success();
    }
    
    @Override
    public String getSystemName() {
        return "alipay";
    }
}

public class ExternalResponse {
    private boolean success;
    private String code;
    private String message;
    private Object data;
    
    public static ExternalResponse success() {
        return success(null);
    }
    
    public static ExternalResponse success(Object data) {
        ExternalResponse response = new ExternalResponse();
        response.setSuccess(true);
        response.setCode("0000");
        response.setMessage("成功");
        response.setData(data);
        return response;
    }
    
    public static ExternalResponse error(String message) {
        ExternalResponse response = new ExternalResponse();
        response.setSuccess(false);
        response.setCode("9999");
        response.setMessage(message);
        return response;
    }
}
                    
                

                
                    
                        
                            
                        
                        支付模块
                    
                    
                        提供多渠道支付接入，包括支付宝、微信支付、银联等，支持支付、查询、退款、对账等功能。
                    
                    
                        @Service
public class PaymentService {
    
    @Autowired
    private PaymentChannelFactory paymentChannelFactory;
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    public PaymentResponse pay(PaymentRequest request) {
        Order order = orderService.getOrder(request.getOrderId());
        
        if (order == null) {
            throw new BusinessException("订单不存在");
        }
        
        if (order.isPaid()) {
            throw new BusinessException("订单已支付");
        }
        
        PaymentChannel channel = paymentChannelFactory.getChannel(request.getChannel());
        
        Payment payment = Payment.builder()
            .orderId(request.getOrderId())
            .amount(order.getAmount())
            .channel(request.getChannel())
            .status(PaymentStatus.PENDING)
            .createTime(new Date())
            .build();
        
        paymentRepository.save(payment);
        
        PaymentChannelResponse channelResponse = channel.pay(
            payment.getId(),
            order.getAmount(),
            request.getExtra()
        );
        
        payment.setThirdPartyPaymentId(channelResponse.getThirdPartyPaymentId());
        paymentRepository.save(payment);
        
        return PaymentResponse.builder()
            .paymentId(payment.getId())
            .paymentUrl(channelResponse.getPaymentUrl())
            .qrCode(channelResponse.getQrCode())
            .build();
    }
    
    public boolean callback(String channel, String paymentId, Map params) {
        PaymentChannel paymentChannel = paymentChannelFactory.getChannel(channel);
        
        if (!paymentChannel.verifyCallback(params)) {
            return false;
        }
        
        Payment payment = paymentRepository.findById(paymentId).orElse(null);
        if (payment == null) {
            return false;
        }
        
        if (payment.getStatus() == PaymentStatus.SUCCESS) {
            return true;
        }
        
        payment.setStatus(PaymentStatus.SUCCESS);
        payment.setPayTime(new Date());
        paymentRepository.save(payment);
        
        orderService.markAsPaid(payment.getOrderId());
        
        return true;
    }
    
    public RefundResponse refund(RefundRequest request) {
        Payment payment = paymentRepository.findById(request.getPaymentId()).orElse(null);
        
        if (payment == null) {
            throw new BusinessException("支付记录不存在");
        }
        
        if (payment.getStatus() != PaymentStatus.SUCCESS) {
            throw new BusinessException("支付未成功，无法退款");
        }
        
        PaymentChannel channel = paymentChannelFactory.getChannel(payment.getChannel());
        
        RefundChannelResponse channelResponse = channel.refund(
            payment.getId(),
            request.getRefundAmount(),
            request.getReason()
        );
        
        Refund refund = Refund.builder()
            .paymentId(payment.getId())
            .refundAmount(request.getRefundAmount())
            .reason(request.getReason())
            .status(RefundStatus.SUCCESS)
            .createTime(new Date())
            .build();
        
        return RefundResponse.builder()
            .refundId(refund.getId())
            .status(refund.getStatus())
            .build();
    }
}

public enum PaymentStatus {
    PENDING,
    SUCCESS,
    FAILED,
    REFUNDED
}
                    
                

                
                    
                        
                            
                        
                        订单模块
                    
                    
                        提供订单全生命周期管理，包括创建、支付、发货、收货、评价、退款等功能。
                    
                    
                        @Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private OrderItemRepository orderItemRepository;
    
    @Autowired
    private IdGenerator idGenerator;
    
    public Order createOrder(CreateOrderRequest request) {
        String orderId = idGenerator.generate();
        
        Order order = Order.builder()
            .orderId(orderId)
            .userId(request.getUserId())
            .totalAmount(request.getTotalAmount())
            .status(OrderStatus.UNPAID)
            .createTime(new Date())
            .build();
        
        List items = request.getItems().stream()
            .map(item -> OrderItem.builder()
                .orderId(orderId)
                .productId(item.getProductId())
                .productName(item.getProductName())
                .quantity(item.getQuantity())
                .price(item.getPrice())
                .build())
            .collect(Collectors.toList());
        
        orderRepository.save(order);
        orderItemRepository.saveAll(items);
        
        return order;
    }
    
    public Order getOrder(String orderId) {
        return orderRepository.findByOrderId(orderId);
    }
    
    public void markAsPaid(String orderId) {
        Order order = orderRepository.findByOrderId(orderId);
        if (order == null) {
            throw new BusinessException("订单不存在");
        }
        
        order.setStatus(OrderStatus.PAID);
        order.setPayTime(new Date());
        orderRepository.save(order);
    }
    
    public void ship(String orderId, ShipRequest request) {
        Order order = orderRepository.findByOrderId(orderId);
        if (order == null) {
            throw new BusinessException("订单不存在");
        }
        
        if (order.getStatus() != OrderStatus.PAID) {
            throw new BusinessException("订单状态不正确");
        }
        
        order.setStatus(OrderStatus.SHIPPED);
        order.setLogisticsCompany(request.getLogisticsCompany());
        order.setLogisticsNo(request.getLogisticsNo());
        order.setShipTime(new Date());
        orderRepository.save(order);
    }
    
    public void confirm(String orderId) {
        Order order = orderRepository.findByOrderId(orderId);
        if (order == null) {
            throw new BusinessException("订单不存在");
        }
        
        if (order.getStatus() != OrderStatus.SHIPPED) {
            throw new BusinessException("订单状态不正确");
        }
        
        order.setStatus(OrderStatus.COMPLETED);
        order.setConfirmTime(new Date());
        orderRepository.save(order);
    }
    
    public void cancel(String orderId, String reason) {
        Order order = orderRepository.findByOrderId(orderId);
        if (order == null) {
            throw new BusinessException("订单不存在");
        }
        
        if (order.getStatus() != OrderStatus.UNPAID && order.getStatus() != OrderStatus.PAID) {
            throw new BusinessException("订单状态不正确");
        }
        
        order.setStatus(OrderStatus.CANCELLED);
        order.setCancelReason(reason);
        order.setCancelTime(new Date());
        orderRepository.save(order);
    }
    
    public List queryOrders(OrderQueryRequest request) {
        return orderRepository.queryByCondition(request);
    }
}

public enum OrderStatus {
    UNPAID("待支付"),
    PAID("已支付"),
    SHIPPED("已发货"),
    COMPLETED("已完成"),
    CANCELLED("已取消"),
    REFUNDED("已退款");
    
    private final String desc;
    
    OrderStatus(String desc) {
        this.desc = desc;
    }
}
                    
                

                
                    
                        
                            
                        
                        客户模块
                    
                    
                        提供客户信息管理、客户画像、客户分级、客户标签、客户跟进等功能。
                    
                    
                        @Service
public class CustomerService {
    
    @Autowired
    private CustomerRepository customerRepository;
    
    @Autowired
    private CustomerTagRepository customerTagRepository;
    
    public Customer createCustomer(CreateCustomerRequest request) {
        Customer customer = Customer.builder()
            .name(request.getName())
            .phone(request.getPhone())
            .email(request.getEmail())
            .company(request.getCompany())
            .level(CustomerLevel.NORMAL)
            .source(request.getSource())
            .createTime(new Date())
            .build();
        
        return customerRepository.save(customer);
    }
    
    public Customer getCustomer(Long customerId) {
        return customerRepository.findById(customerId).orElse(null);
    }
    
    public void updateCustomer(Long customerId, UpdateCustomerRequest request) {
        Customer customer = customerRepository.findById(customerId).orElse(null);
        if (customer == null) {
            throw new BusinessException("客户不存在");
        }
        
        if (request.getName() != null) {
            customer.setName(request.getName());
        }
        if (request.getPhone() != null) {
            customer.setPhone(request.getPhone());
        }
        if (request.getEmail() != null) {
            customer.setEmail(request.getEmail());
        }
        
        customerRepository.save(customer);
    }
    
    public void addTags(Long customerId, List tags) {
        Customer customer = customerRepository.findById(customerId).orElse(null);
        if (customer == null) {
            throw new BusinessException("客户不存在");
        }
        
        List tagList = tags.stream()
            .map(tag -> CustomerTag.builder()
                .customerId(customerId)
                .tag(tag)
                .createTime(new Date())
                .build())
            .collect(Collectors.toList());
        
        customerTagRepository.saveAll(tagList);
    }
    
    public void updateLevel(Long customerId, CustomerLevel level) {
        Customer customer = customerRepository.findById(customerId).orElse(null);
        if (customer == null) {
            throw new BusinessException("客户不存在");
        }
        
        customer.setLevel(level);
        customerRepository.save(customer);
    }
    
    public void addFollow(Long customerId, FollowRequest request) {
        Customer customer = customerRepository.findById(customerId).orElse(null);
        if (customer == null) {
            throw new BusinessException("客户不存在");
        }
        
        CustomerFollow follow = CustomerFollow.builder()
            .customerId(customerId)
            .content(request.getContent())
            .type(request.getType())
            .createTime(new Date())
            .build();
        
        customer.setLastFollowTime(new Date());
        customerRepository.save(customer);
    }
    
    public List queryCustomers(CustomerQueryRequest request) {
        return customerRepository.queryByCondition(request);
    }
    
    public CustomerProfile getProfile(Long customerId) {
        Customer customer = customerRepository.findById(customerId).orElse(null);
        if (customer == null) {
            throw new BusinessException("客户不存在");
        }
        
        List tags = customerTagRepository.findByCustomerId(customerId);
        
        return CustomerProfile.builder()
            .customer(customer)
            .tags(tags.stream().map(CustomerTag::getTag).collect(Collectors.toList()))
            .build();
    }
}

public enum CustomerLevel {
    NORMAL("普通客户"),
    SILVER("白银客户"),
    GOLD("黄金客户"),
    PLATINUM("铂金客户"),
    DIAMOND("钻石客户");
    
    private final String desc;
    
    CustomerLevel(String desc) {
        this.desc = desc;
    }
}
                    
                

                
                    
                        
                            
                        
                        智能体模块
                    
                    
                        提供AI智能助手功能，包括对话、知识库、意图识别、自动回复等功能。
                    
                    
                        @Service
public class AgentService {
    
    @Autowired
    private LlmService llmService;
    
    @Autowired
    private KnowledgeBaseService knowledgeBaseService;
    
    @Autowired
    private ConversationRepository conversationRepository;
    
    @Autowired
    private MessageRepository messageRepository;
    
    public ChatResponse chat(ChatRequest request) {
        String conversationId = request.getConversationId();
        
        if (conversationId == null) {
            conversationId = createConversation(request.getUserId());
        }
        
        saveMessage(conversationId, MessageRole.USER, request.getContent());
        
        List history = messageRepository.findByConversationId(conversationId);
        
        String context = buildContext(history);
        
        List knowledge = knowledgeBaseService.retrieve(request.getContent(), 3);
        
        String prompt = buildPrompt(request.getContent(), context, knowledge);
        
        String answer = llmService.generate(prompt);
        
        saveMessage(conversationId, MessageRole.ASSISTANT, answer);
        
        return ChatResponse.builder()
            .conversationId(conversationId)
            .content(answer)
            .build();
    }
    
    private String createConversation(Long userId) {
        Conversation conversation = Conversation.builder()
            .userId(userId)
            .createTime(new Date())
            .build();
        
        conversationRepository.save(conversation);
        return conversation.getId();
    }
    
    private void saveMessage(String conversationId, MessageRole role, String content) {
        Message message = Message.builder()
            .conversationId(conversationId)
            .role(role)
            .content(content)
            .createTime(new Date())
            .build();
        
        messageRepository.save(message);
    }
    
    private String buildContext(List history) {
        StringBuilder context = new StringBuilder();
        for (Message message : history) {
            context.append(message.getRole().name().toLowerCase())
                   .append(": ")
                   .append(message.getContent())
                   .append("\n");
        }
        return context.toString();
    }
    
    private String buildPrompt(String query, String context, List knowledge) {
        StringBuilder prompt = new StringBuilder();
        prompt.append("你是一个智能助手，请根据以下信息回答用户问题。\n\n");
        
        if (!knowledge.isEmpty()) {
            prompt.append("参考知识库:\n");
            for (String k : knowledge) {
                prompt.append("- ").append(k).append("\n");
            }
            prompt.append("\n");
        }
        
        if (!context.isEmpty()) {
            prompt.append("对话历史:\n").append(context).append("\n");
        }
        
        prompt.append("用户问题: ").append(query).append("\n");
        prompt.append("请回答:");
        
        return prompt.toString();
    }
    
    public List getConversations(Long userId) {
        return conversationRepository.findByUserId(userId);
    }
    
    public List getMessages(String conversationId) {
        return messageRepository.findByConversationId(conversationId);
    }
    
    public void addToKnowledgeBase(KnowledgeRequest request) {
        Knowledge knowledge = Knowledge.builder()
            .title(request.getTitle())
            .content(request.getContent())
            .category(request.getCategory())
            .createTime(new Date())
            .build();
        
        knowledgeBaseService.add(knowledge);
    }
}

public enum MessageRole {
    USER,
    ASSISTANT,
    SYSTEM
}

@Service
public class KnowledgeBaseService {
    
    @Autowired
    private VectorDatabase vectorDatabase;
    
    public void add(Knowledge knowledge) {
        String embedding = generateEmbedding(knowledge.getContent());
        vectorDatabase.store(knowledge.getId(), embedding, knowledge);
    }
    
    public List retrieve(String query, int topK) {
        String queryEmbedding = generateEmbedding(query);
        List results = vectorDatabase.search(queryEmbedding, topK);
        return results.stream().map(Knowledge::getContent).collect(Collectors.toList());
    }
    
    private String generateEmbedding(String text) {
        return "";
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    模块化设计
                    高内聚低耦合，易于扩展
                
                
                    
                    适配器模式
                    灵活接入第三方系统
                
                
                    
                    复用性强
                    统一服务，减少重复开发
                
                
                    
                    安全可靠
                    完整的安全防护机制
                
                
                    
                    高性能
                    缓存优化，异步处理
                
                
                    
                    智能赋能
                    AI智能体提升效率
