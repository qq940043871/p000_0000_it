消息系统 - 架构师学习笔记
    
    


    
        
            
                消息系统设计
            
            
            
                
                    消息系统是社交平台中不可或缺的组件，负责处理用户间的实时通信。一个优秀的消息系统需要具备高并发、低延迟、高可靠性等特点。
                
                
                系统架构
                
                
                    
                        消息系统通常采用发布/订阅模式，结合长连接和消息队列技术实现。
                    
                
                
                核心组件
                
                    消息网关：负责连接管理、协议转换、负载均衡
                    消息存储：持久化消息内容，支持历史消息查询
                    消息队列：异步处理消息投递，削峰填谷
                    推送服务：负责消息的实时推送
                    离线服务：处理离线消息存储与推送
                
                
                技术实现
                
                长连接方案
                
// 使用Netty实现WebSocket长连接
@Component
public class WebSocketServer {
    private final ChannelGroup channels = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    
    public void handleConnection(WebSocketSession session) {
        // 添加连接到连接组
        channels.add(session.getChannel());
        
        // 注册用户与连接的映射关系
        UserConnectionManager.register(session.getUserId(), session.getChannel());
    }
    
    public void handleMessage(WebSocketSession session, TextWebSocketFrame frame) {
        Message message = JSON.parseObject(frame.text(), Message.class);
        
        // 消息处理逻辑
        messageService.processMessage(message);
    }
}
                
                消息存储设计
                
// 消息表设计
CREATE TABLE message (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sender_id BIGINT NOT NULL,
    receiver_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    msg_type TINYINT NOT NULL DEFAULT 1, -- 1:文本 2:图片 3:语音
    status TINYINT NOT NULL DEFAULT 0,   -- 0:未读 1:已读
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_sender_receiver (sender_id, receiver_id, create_time),
    INDEX idx_receiver_time (receiver_id, create_time)
);

// 使用Redis缓存最近消息
String redisKey = "recent_messages:" + userId;
redisTemplate.opsForList().leftPush(redisKey, message);
redisTemplate.expire(redisKey, Duration.ofHours(24));
                
                消息投递保证
                
@Service
public class MessageDeliveryService {
    
    @Autowired
    private MessageQueue messageQueue;
    
    @Autowired
    private UserConnectionManager connectionManager;
    
    public void sendMessage(Message message) {
        // 1. 持久化消息
        messageRepository.save(message);
        
        // 2. 推送到消息队列
        messageQueue.send("message_delivery", message);
        
        // 3. 实时推送（如果用户在线）
        Channel channel = connectionManager.getChannel(message.getReceiverId());
        if (channel != null && channel.isActive()) {
            channel.writeAndFlush(new TextWebSocketFrame(JSON.toJSONString(message)));
        }
    }
    
    @RabbitListener(queues = "message_delivery")
    public void handleMessageDelivery(Message message) {
        // 异步处理消息投递
        Channel channel = connectionManager.getChannel(message.getReceiverId());
        if (channel != null && channel.isActive()) {
            // 在线投递
            channel.writeAndFlush(new TextWebSocketFrame(JSON.toJSONString(message)));
        } else {
            // 离线存储
            offlineMessageService.save(message);
        }
    }
}
                
                性能优化
                
                连接管理优化
                
                    心跳机制：定期发送心跳包维持连接
                    连接复用：同一用户多个设备共享连接
                    负载均衡：根据连接数和CPU使用率分配连接
                    连接迁移：服务器重启时平滑迁移连接
                
                
                消息处理优化
                
                    批量处理：合并多个消息统一处理
                    异步处理：使用线程池异步处理耗时操作
                    缓存热点：缓存高频用户的消息内容
                    压缩传输：对消息内容进行压缩传输
                
                
                可靠性保障
                
                消息可靠性
                
                    ACK机制：接收方确认收到消息
                    重试机制：失败消息自动重试
                    消息去重：防止重复消息
                    顺序保证：保证同一会话消息顺序
                
                
                系统监控
                
                    连接监控：实时监控连接数和活跃度
                    延迟监控：监控消息投递延迟
                    错误监控：监控系统异常和错误
                    容量规划：根据监控数据进行容量规划
                
                
                
                    最佳实践
                    
                        使用成熟的MQ组件如RabbitMQ、Kafka处理异步消息
                        合理设计消息表索引，优化查询性能
                        实现消息确认机制，确保消息不丢失
                        对热点用户进行限流保护
                        建立完善的监控告警机制
