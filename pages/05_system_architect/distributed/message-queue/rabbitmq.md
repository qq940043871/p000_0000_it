RabbitMQ - 架构师学习笔记
    
    


    
        
            
                RabbitMQ
            
            
            
                
                    RabbitMQ是一个开源的消息代理和队列服务器，用来实现应用程序之间的异步通信。它基于AMQP（高级消息队列协议）实现，支持多种消息传递模式，具有高可靠性、灵活的路由、集群和高可用性等特性。
                
                
                RabbitMQ核心概念
                
                
                    
                        
                            
                            Producer（生产者）
                        
                        
                            消息的发送方，负责创建消息并将其发送到RabbitMQ Broker。
                        
                    
                    
                    
                        
                            
                            Consumer（消费者）
                        
                        
                            消息的接收方，从RabbitMQ Broker获取消息并进行处理。
                        
                    
                    
                    
                        
                            
                            Broker（消息代理）
                        
                        
                            消息中间件服务器，负责接收、存储和转发消息。
                        
                    
                    
                    
                        
                            
                            Exchange（交换机）
                        
                        
                            接收生产者发送的消息，并根据路由规则将消息分发到队列中。
                        
                    
                    
                    
                        
                            
                            Queue（队列）
                        
                        
                            存储消息的缓冲区，消息在队列中等待消费者处理。
                        
                    
                    
                    
                        
                            
                            Binding（绑定）
                        
                        
                            连接Exchange和Queue的规则，定义了消息如何从Exchange路由到Queue。
                        
                    
                
                
                Exchange类型
                
                
                    
                        
                            
                                类型
                                路由规则
                                特点
                            
                        
                        
                            
                                Direct
                                精确匹配Routing Key
                                点对点通信
                            
                            
                                Fanout
                                广播模式，忽略Routing Key
                                发布订阅模式
                            
                            
                                Topic
                                模式匹配Routing Key
                                通配符匹配
                            
                            
                                Headers
                                匹配消息头部属性
                                忽略Routing Key
                            
                        
                    
                
                
                消息传递模式
                
                
                    
                        
                            
                            简单模式（Simple）
                        
                        
                            最简单的模式，一个生产者对应一个消费者。
                        
                        
// 生产者
channel.queueDeclare("hello", false, false, false, null);
channel.basicPublish("", "hello", null, message.getBytes());

// 消费者
channel.basicConsume("hello", true, consumer);
                    
                    
                    
                        
                            
                            工作队列模式（Work Queue）
                        
                        
                            一个生产者，多个消费者，消息轮流分发给消费者。
                        
                        
// 生产者
channel.queueDeclare("task_queue", true, false, false, null);
channel.basicPublish("", "task_queue", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());

// 消费者
channel.basicQos(1); // 公平分发
channel.basicConsume("task_queue", false, consumer);
                    
                    
                    
                        
                            
                            发布/订阅模式（Publish/Subscribe）
                        
                        
                            一个生产者，多个消费者，消息广播给所有绑定的队列。
                        
                        
// 生产者
channel.exchangeDeclare("logs", "fanout");
channel.basicPublish("logs", "", null, message.getBytes());

// 消费者
channel.exchangeDeclare("logs", "fanout");
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, "logs", "");
channel.basicConsume(queueName, true, consumer);
                    
                    
                    
                        
                            
                            路由模式（Routing）
                        
                        
                            根据Routing Key精确匹配，将消息分发到不同的队列。
                        
                        
// 生产者
channel.exchangeDeclare("direct_logs", "direct");
channel.basicPublish("direct_logs", severity, null, message.getBytes());

// 消费者
channel.exchangeDeclare("direct_logs", "direct");
String queueName = channel.queueDeclare().getQueue();
channel.queueBind(queueName, "direct_logs", "error");
channel.queueBind(queueName, "direct_logs", "warning");
channel.basicConsume(queueName, true, consumer);
                    
                
                
                高级特性
                
                
                    
                        
                            
                            消息确认机制
                        
                        
                            确保消息可靠传递，防止消息丢失。
                        
                        
                            生产者确认：确认消息已到达Broker
                            消费者确认：确认消息已被消费
                        
                    
                    
                    
                        
                            
                            消息持久化
                        
                        
                            将消息存储到磁盘，防止Broker重启后消息丢失。
                        
                        
                            队列持久化：设置队列durable为true
                            消息持久化：设置消息deliveryMode为2
                        
                    
                    
                    
                        
                            
                            公平分发
                        
                        
                            避免某些消费者过载，实现负载均衡。
                        
                        
channel.basicQos(1); // 每次只处理一条消息
channel.basicConsume(queueName, false, consumer); // 手动确认
                    
                    
                    
                        
                            
                            死信队列
                        
                        
                            处理无法被正常消费的消息。
                        
                        
                            消息被拒绝且requeue为false
                            消息过期
                            队列达到最大长度
                        
                    
                
                
                集群与高可用
                
                
                    集群模式
                    
                        
                            普通集群模式
                            
                                同步元数据，不同步队列数据
                            
                        
                        
                            镜像队列模式
                            
                                同步队列数据到多个节点
                            
                        
                        
                            仲裁队列模式
                            
                                新一代高可用队列，基于Raft协议
                            
                        
                    
                    
                    高可用配置
                    
# 镜像队列配置
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'

# 仲裁队列配置
channel.queueDeclare("my-queue", true, false, false, 
    Collections.singletonMap("x-queue-type", "quorum"));
                
                
                监控与管理
                
                
                    
                        
                            
                            Web管理界面
                        
                        
                            RabbitMQ提供Web管理界面，默认端口15672。
                        
                        
                            查看队列、交换机、绑定等信息
                            监控消息流量和性能指标
                            管理用户和权限
                            查看节点状态和集群信息
                        
                    
                    
                    
                        
                            
                            命令行工具
                        
                        
                            rabbitmqctl是RabbitMQ的命令行管理工具。
                        
                        
# 查看队列列表
rabbitmqctl list_queues

# 查看交换机列表
rabbitmqctl list_exchanges

# 查看绑定关系
rabbitmqctl list_bindings

# 添加用户
rabbitmqctl add_user username password
                    
                
                
                最佳实践
                
                    
                        合理设计Exchange和Queue，避免过多的队列和绑定
                        使用消息确认机制确保消息不丢失
                        设置合适的消息过期时间和队列长度限制
                        实施监控和告警，及时发现和处理异常
                        定期清理无用的队列和交换机
                        使用集群和镜像队列提高可用性
                        合理设置资源限制，防止消息积压
