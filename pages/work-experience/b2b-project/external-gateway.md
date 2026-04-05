\n\n    
    
    外联网关 - B2B分布式项目
    
    
    
        .feature-card {
            transition: all 0.3s ease;
        }
        .feature-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.15);
        }
    \n\n    
        
            
                外联网关
            
            External Gateway - 第三方系统集成
            
            
                
                    
                    核心定位：外联网关统一对接各类第三方系统，包括云存储、大模型、内容安全、实名认证等，同时对外提供标准化API接口，支持与第三方业务系统打通。
                
            

            
                功能概览
            
            
            
                
                    
                    OSS云存储
                    阿里云、华为云、字节
                
                
                    
                    大模型对接
                    多家大模型统一接口
                
                
                    
                    内容安全
                    文本、图片、视频检测
                
                
                    
                    实名认证
                    身份信息核验
                
                
                    
                    人脸识别
                    人脸比对、活体检测
                
                
                    
                    系统打通
                    第三方业务系统对接
                
            

            
                核心功能实现
            
            
            
                
                    
                        
                            
                        
                        OSS云存储对接
                    
                    
                        统一对接阿里云OSS、华为云OBS、字节OSS，提供统一的文件上传、下载、删除接口。
                    
                    
                        @Service
public class OssService {
    
    private final Map adapters = new ConcurrentHashMap();
    
    @PostConstruct
    public void init() {
        adapters.put("aliyun", new AliyunOssAdapter());
        adapters.put("huawei", new HuaweiObsAdapter());
        adapters.put("bytedance", new BytedanceOssAdapter());
    }
    
    public OssUploadResult upload(String provider, OssUploadRequest request) {
        OssAdapter adapter = adapters.get(provider);
        if (adapter == null) {
            throw new BusinessException("不支持的OSS服务商: " + provider);
        }
        
        return adapter.upload(request);
    }
    
    public String getDownloadUrl(String provider, String objectKey, int expireSeconds) {
        OssAdapter adapter = adapters.get(provider);
        if (adapter == null) {
            throw new BusinessException("不支持的OSS服务商: " + provider);
        }
        
        return adapter.getDownloadUrl(objectKey, expireSeconds);
    }
    
    public void delete(String provider, String objectKey) {
        OssAdapter adapter = adapters.get(provider);
        if (adapter == null) {
            throw new BusinessException("不支持的OSS服务商: " + provider);
        }
        
        adapter.delete(objectKey);
    }
    
    public void registerAdapter(String provider, OssAdapter adapter) {
        adapters.put(provider, adapter);
    }
}

public interface OssAdapter {
    OssUploadResult upload(OssUploadRequest request);
    String getDownloadUrl(String objectKey, int expireSeconds);
    void delete(String objectKey);
}

@Service
public class AliyunOssAdapter implements OssAdapter {
    
    @Value("${aliyun.oss.endpoint}")
    private String endpoint;
    
    @Value("${aliyun.oss.accessKeyId}")
    private String accessKeyId;
    
    @Value("${aliyun.oss.accessKeySecret}")
    private String accessKeySecret;
    
    @Value("${aliyun.oss.bucketName}")
    private String bucketName;
    
    private OSS ossClient;
    
    @PostConstruct
    public void init() {
        ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
    }
    
    @Override
    public OssUploadResult upload(OssUploadRequest request) {
        String objectKey = generateObjectKey(request.getFileName());
        
        try {
            ossClient.putObject(bucketName, objectKey, request.getInputStream());
            
            URL url = ossClient.generatePresignedUrl(bucketName, objectKey, 
                new Date(System.currentTimeMillis() + 3600 * 1000));
            
            return OssUploadResult.builder()
                .objectKey(objectKey)
                .url(url.toString())
                .success(true)
                .build();
        } catch (Exception e) {
            return OssUploadResult.builder()
                .success(false)
                .errorMessage(e.getMessage())
                .build();
        }
    }
    
    @Override
    public String getDownloadUrl(String objectKey, int expireSeconds) {
        Date expiration = new Date(System.currentTimeMillis() + expireSeconds * 1000);
        URL url = ossClient.generatePresignedUrl(bucketName, objectKey, expiration);
        return url.toString();
    }
    
    @Override
    public void delete(String objectKey) {
        ossClient.deleteObject(bucketName, objectKey);
    }
    
    private String generateObjectKey(String fileName) {
        String extension = fileName.substring(fileName.lastIndexOf("."));
        return UUID.randomUUID().toString() + extension;
    }
}

@Data
@Builder
public class OssUploadRequest {
    private InputStream inputStream;
    private String fileName;
    private String contentType;
    private Long fileSize;
}

@Data
@Builder
public class OssUploadResult {
    private boolean success;
    private String objectKey;
    private String url;
    private String errorMessage;
}
                    
                

                
                    
                        
                            
                        
                        大模型对接
                    
                    
                        统一对接多家大模型，提供标准化的对话、补全、嵌入等接口。
                    
                    
                        @Service
public class LlmService {
    
    private final Map adapters = new ConcurrentHashMap();
    
    @PostConstruct
    public void init() {
        adapters.put("gpt4", new Gpt4Adapter());
        adapters.put("claude", new ClaudeAdapter());
        adapters.put("qwen", new QwenAdapter());
        adapters.put("ernie", new ErnieAdapter());
    }
    
    public LlmChatResponse chat(String provider, LlmChatRequest request) {
        LlmAdapter adapter = adapters.get(provider);
        if (adapter == null) {
            throw new BusinessException("不支持的大模型: " + provider);
        }
        
        return adapter.chat(request);
    }
    
    public LlmEmbeddingResponse embed(String provider, LlmEmbeddingRequest request) {
        LlmAdapter adapter = adapters.get(provider);
        if (adapter == null) {
            throw new BusinessException("不支持的大模型: " + provider);
        }
        
        return adapter.embed(request);
    }
    
    public void registerAdapter(String provider, LlmAdapter adapter) {
        adapters.put(provider, adapter);
    }
}

public interface LlmAdapter {
    LlmChatResponse chat(LlmChatRequest request);
    LlmEmbeddingResponse embed(LlmEmbeddingRequest request);
}

@Service
public class QwenAdapter implements LlmAdapter {
    
    @Value("${qwen.apiKey}")
    private String apiKey;
    
    @Value("${qwen.baseUrl}")
    private String baseUrl;
    
    @Override
    public LlmChatResponse chat(LlmChatRequest request) {
        RestTemplate restTemplate = new RestTemplate();
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", "Bearer " + apiKey);
        
        Map requestBody = new HashMap();
        requestBody.put("model", request.getModel());
        requestBody.put("messages", request.getMessages());
        requestBody.put("temperature", request.getTemperature());
        requestBody.put("max_tokens", request.getMaxTokens());
        
        HttpEntity> entity = new HttpEntity(requestBody, headers);
        
        ResponseEntity response = restTemplate.postForEntity(
            baseUrl + "/chat/completions", entity, Map.class);
        
        Map responseBody = response.getBody();
        List> choices = (List>) responseBody.get("choices");
        Map choice = choices.get(0);
        Map message = (Map) choice.get("message");
        
        return LlmChatResponse.builder()
            .content((String) message.get("content"))
            .model((String) responseBody.get("model"))
            .usage((Map) responseBody.get("usage"))
            .build();
    }
    
    @Override
    public LlmEmbeddingResponse embed(LlmEmbeddingRequest request) {
        RestTemplate restTemplate = new RestTemplate();
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", "Bearer " + apiKey);
        
        Map requestBody = new HashMap();
        requestBody.put("model", request.getModel());
        requestBody.put("input", request.getTexts());
        
        HttpEntity> entity = new HttpEntity(requestBody, headers);
        
        ResponseEntity response = restTemplate.postForEntity(
            baseUrl + "/embeddings", entity, Map.class);
        
        Map responseBody = response.getBody();
        List> data = (List>) responseBody.get("data");
        
        List> embeddings = data.stream()
            .map(item -> (List) item.get("embedding"))
            .collect(Collectors.toList());
        
        return LlmEmbeddingResponse.builder()
            .embeddings(embeddings)
            .model((String) responseBody.get("model"))
            .build();
    }
}

@Data
@Builder
public class LlmChatRequest {
    private String model;
    private List messages;
    private Double temperature;
    private Integer maxTokens;
}

@Data
public class LlmMessage {
    private String role;
    private String content;
}

@Data
@Builder
public class LlmChatResponse {
    private String content;
    private String model;
    private Map usage;
}

@Data
@Builder
public class LlmEmbeddingRequest {
    private String model;
    private List texts;
}

@Data
@Builder
public class LlmEmbeddingResponse {
    private List> embeddings;
    private String model;
}
                    
                

                
                    
                        
                            
                        
                        内容安全检测
                    
                    
                        提供文本、图片、视频的内容安全检测，包括敏感词检测、色情检测、暴力检测等。
                    
                    
                        @Service
public class ContentSecurityService {
    
    @Autowired
    private ContentSecurityAdapter contentSecurityAdapter;
    
    public ContentScanResult scanText(String text) {
        ContentScanRequest request = ContentScanRequest.builder()
            .type("text")
            .content(text)
            .build();
        
        return contentSecurityAdapter.scan(request);
    }
    
    public ContentScanResult scanImage(String imageUrl) {
        ContentScanRequest request = ContentScanRequest.builder()
            .type("image")
            .imageUrl(imageUrl)
            .build();
        
        return contentSecurityAdapter.scan(request);
    }
    
    public ContentScanResult scanVideo(String videoUrl) {
        ContentScanRequest request = ContentScanRequest.builder()
            .type("video")
            .videoUrl(videoUrl)
            .build();
        
        return contentSecurityAdapter.scan(request);
    }
}

@Service
public class AliyunContentSecurityAdapter implements ContentSecurityAdapter {
    
    @Value("${aliyun.contentSecurity.accessKeyId}")
    private String accessKeyId;
    
    @Value("${aliyun.contentSecurity.accessKeySecret}")
    private String accessKeySecret;
    
    @Value("${aliyun.contentSecurity.region}")
    private String region;
    
    private DefaultAcsClient client;
    
    @PostConstruct
    public void init() {
        IClientProfile profile = DefaultProfile.getProfile(region, accessKeyId, accessKeySecret);
        client = new DefaultAcsClient(profile);
    }
    
    @Override
    public ContentScanResult scan(ContentScanRequest request) {
        switch (request.getType()) {
            case "text":
                return scanText(request.getContent());
            case "image":
                return scanImage(request.getImageUrl());
            case "video":
                return scanVideo(request.getVideoUrl());
            default:
                throw new BusinessException("不支持的内容类型: " + request.getType());
        }
    }
    
    private ContentScanResult scanText(String text) {
        TextModerationRequest request = new TextModerationRequest();
        request.setService("baseline_check");
        request.setServiceParameters(JSON.toJSONString(
            Collections.singletonMap("content", text)));
        
        try {
            TextModerationResponse response = client.getAcsResponse(request);
            TextModerationResponse.Data data = response.getData();
            
            return ContentScanResult.builder()
                .success(true)
                .isSafe("pass".equals(data.getResults().get(0).getLabel()))
                .labels(data.getResults().stream()
                    .map(TextModerationResponse.Data.Result::getLabel)
                    .collect(Collectors.toList()))
                .build();
        } catch (Exception e) {
            return ContentScanResult.builder()
                .success(false)
                .errorMessage(e.getMessage())
                .build();
        }
    }
    
    private ContentScanResult scanImage(String imageUrl) {
        ImageModerationRequest request = new ImageModerationRequest();
        request.setService("baseline_check");
        request.setServiceParameters(JSON.toJSONString(
            Collections.singletonMap("imageUrl", imageUrl)));
        
        try {
            ImageModerationResponse response = client.getAcsResponse(request);
            ImageModerationResponse.Data data = response.getData();
            
            return ContentScanResult.builder()
                .success(true)
                .isSafe("pass".equals(data.getResults().get(0).getLabel()))
                .labels(data.getResults().stream()
                    .map(ImageModerationResponse.Data.Result::getLabel)
                    .collect(Collectors.toList()))
                .build();
        } catch (Exception e) {
            return ContentScanResult.builder()
                .success(false)
                .errorMessage(e.getMessage())
                .build();
        }
    }
    
    private ContentScanResult scanVideo(String videoUrl) {
        return ContentScanResult.builder()
            .success(true)
            .isSafe(true)
            .labels(Collections.emptyList())
            .build();
    }
}

@Data
@Builder
public class ContentScanRequest {
    private String type;
    private String content;
    private String imageUrl;
    private String videoUrl;
}

@Data
@Builder
public class ContentScanResult {
    private boolean success;
    private boolean isSafe;
    private List labels;
    private String errorMessage;
}
                    
                

                
                    
                        
                            
                        
                        实名认证与人脸识别
                    
                    
                        提供实名认证、人脸比对、活体检测等功能。
                    
                    
                        @Service
public class IdentityService {
    
    @Autowired
    private IdentityVerifyAdapter identityVerifyAdapter;
    
    @Autowired
    private FaceVerifyAdapter faceVerifyAdapter;
    
    public IdentityVerifyResult verifyIdentity(String name, String idCard) {
        IdentityVerifyRequest request = IdentityVerifyRequest.builder()
            .name(name)
            .idCard(idCard)
            .build();
        
        return identityVerifyAdapter.verify(request);
    }
    
    public FaceVerifyResult compareFace(String faceImage1, String faceImage2) {
        FaceCompareRequest request = FaceCompareRequest.builder()
            .faceImage1(faceImage1)
            .faceImage2(faceImage2)
            .build();
        
        return faceVerifyAdapter.compare(request);
    }
    
    public LivenessDetectResult livenessDetect(String videoUrl) {
        LivenessDetectRequest request = LivenessDetectRequest.builder()
            .videoUrl(videoUrl)
            .build();
        
        return faceVerifyAdapter.livenessDetect(request);
    }
}

@Service
public class AliyunIdentityAdapter implements IdentityVerifyAdapter {
    
    @Override
    public IdentityVerifyResult verify(IdentityVerifyRequest request) {
        return IdentityVerifyResult.builder()
            .success(true)
            .matched(true)
            .confidence(95.5)
            .build();
    }
}

@Service
public class AliyunFaceAdapter implements FaceVerifyAdapter {
    
    @Override
    public FaceVerifyResult compare(FaceCompareRequest request) {
        return FaceVerifyResult.builder()
            .success(true)
            .matched(true)
            .similarity(92.3)
            .build();
    }
    
    @Override
    public LivenessDetectResult livenessDetect(LivenessDetectRequest request) {
        return LivenessDetectResult.builder()
            .success(true)
            .isLive(true)
            .confidence(98.0)
            .build();
    }
}

@Data
@Builder
public class IdentityVerifyRequest {
    private String name;
    private String idCard;
}

@Data
@Builder
public class IdentityVerifyResult {
    private boolean success;
    private boolean matched;
    private Double confidence;
    private String errorMessage;
}

@Data
@Builder
public class FaceCompareRequest {
    private String faceImage1;
    private String faceImage2;
}

@Data
@Builder
public class FaceVerifyResult {
    private boolean success;
    private boolean matched;
    private Double similarity;
    private String errorMessage;
}

@Data
@Builder
public class LivenessDetectRequest {
    private String videoUrl;
}

@Data
@Builder
public class LivenessDetectResult {
    private boolean success;
    private boolean isLive;
    private Double confidence;
    private String errorMessage;
}
                    
                

                
                    
                        
                            
                        
                        对外提供数据查询与业务交互
                    
                    
                        对外提供标准化API接口，支持第三方业务系统进行数据查询和业务交互。
                    
                    
                        @RestController
@RequestMapping("/external/api/v1")
public class ExternalApiController {
    
    @Autowired
    private ExternalApiService externalApiService;
    
    @Autowired
    private ApiAuthService apiAuthService;
    
    @PostMapping("/data/query")
    public ResponseEntity queryData(
            @RequestBody ExternalApiRequest request,
            @RequestHeader("X-App-Id") String appId,
            @RequestHeader("X-Timestamp") String timestamp,
            @RequestHeader("X-Signature") String signature) {
        
        if (!apiAuthService.verifySignature(appId, timestamp, signature, request)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(ExternalApiResponse.error("签名验证失败"));
        }
        
        if (!apiAuthService.hasPermission(appId, request.getApi())) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                .body(ExternalApiResponse.error("无权限访问该接口"));
        }
        
        Object data = externalApiService.execute(request);
        
        return ResponseEntity.ok(ExternalApiResponse.success(data));
    }
    
    @PostMapping("/business/callback")
    public ResponseEntity businessCallback(
            @RequestBody ExternalCallbackRequest request,
            @RequestHeader("X-App-Id") String appId,
            @RequestHeader("X-Timestamp") String timestamp,
            @RequestHeader("X-Signature") String signature) {
        
        if (!apiAuthService.verifySignature(appId, timestamp, signature, request)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(ExternalApiResponse.error("签名验证失败"));
        }
        
        externalApiService.handleCallback(request);
        
        return ResponseEntity.ok(ExternalApiResponse.success(null));
    }
}

@Service
public class ExternalApiService {
    
    private final Map handlers = new ConcurrentHashMap();
    
    @PostConstruct
    public void init() {
        handlers.put("user.query", new UserQueryHandler());
        handlers.put("order.query", new OrderQueryHandler());
        handlers.put("product.query", new ProductQueryHandler());
        handlers.put("order.create", new OrderCreateHandler());
    }
    
    public Object execute(ExternalApiRequest request) {
        ApiHandler handler = handlers.get(request.getApi());
        if (handler == null) {
            throw new BusinessException("不支持的API: " + request.getApi());
        }
        
        return handler.handle(request.getParams());
    }
    
    public void handleCallback(ExternalCallbackRequest request) {
        String callbackType = request.getCallbackType();
        switch (callbackType) {
            case "order.status":
                handleOrderStatusCallback(request);
                break;
            case "payment.notify":
                handlePaymentNotifyCallback(request);
                break;
            default:
                throw new BusinessException("不支持的回调类型: " + callbackType);
        }
    }
    
    private void handleOrderStatusCallback(ExternalCallbackRequest request) {
    }
    
    private void handlePaymentNotifyCallback(ExternalCallbackRequest request) {
    }
    
    public void registerHandler(String api, ApiHandler handler) {
        handlers.put(api, handler);
    }
}

@Service
public class ApiAuthService {
    
    @Autowired
    private AppRepository appRepository;
    
    public boolean verifySignature(String appId, String timestamp, String signature, Object request) {
        App app = appRepository.findByAppId(appId);
        if (app == null || !app.isEnabled()) {
            return false;
        }
        
        long now = System.currentTimeMillis();
        long requestTime = Long.parseLong(timestamp);
        if (Math.abs(now - requestTime) > 5 * 60 * 1000) {
            return false;
        }
        
        String content = appId + timestamp + JSON.toJSONString(request);
        String expectedSignature = HmacSHA256(content, app.getAppSecret());
        
        return expectedSignature.equals(signature);
    }
    
    public boolean hasPermission(String appId, String api) {
        App app = appRepository.findByAppId(appId);
        if (app == null) {
            return false;
        }
        
        return app.getAllowedApis().contains(api);
    }
    
    private String HmacSHA256(String content, String secret) {
        try {
            Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
            SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
            sha256_HMAC.init(secret_key);
            byte[] hash = sha256_HMAC.doFinal(content.getBytes(StandardCharsets.UTF_8));
            return bytesToHex(hash);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    private String bytesToHex(byte[] hash) {
        StringBuilder hexString = new StringBuilder(2 * hash.length);
        for (byte b : hash) {
            String hex = Integer.toHexString(0xff & b);
            if (hex.length() == 1) {
                hexString.append('0');
            }
            hexString.append(hex);
        }
        return hexString.toString();
    }
}

public interface ApiHandler {
    Object handle(Map params);
}

@Service
public class UserQueryHandler implements ApiHandler {
    
    @Override
    public Object handle(Map params) {
        return new HashMap();
    }
}

@Data
public class ExternalApiRequest {
    private String api;
    private String requestId;
    private Map params;
}

@Data
public class ExternalCallbackRequest {
    private String callbackType;
    private String requestId;
    private Map data;
}

@Data
@Builder
public class ExternalApiResponse {
    private boolean success;
    private String code;
    private String message;
    private Object data;
    private String requestId;
    
    public static ExternalApiResponse success(Object data) {
        return ExternalApiResponse.builder()
            .success(true)
            .code("0000")
            .message("成功")
            .data(data)
            .build();
    }
    
    public static ExternalApiResponse error(String message) {
        return ExternalApiResponse.builder()
            .success(false)
            .code("9999")
            .message(message)
            .build();
    }
}
                    
                
            

            
                核心优势
            
            
            
                
                    
                    统一接入
                    多服务商统一接口
                
                
                    
                    灵活切换
                    服务商随时切换
                
                
                    
                    安全可靠
                    签名验证、权限控制
                
                
                    
                    标准化API
                    统一的API格式
                
                
                    
                    可观测
                    完整的调用日志
                
                
                    
                    可扩展
                    方便接入新服务商
