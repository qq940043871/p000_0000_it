订单系统 - 架构师学习笔记
    
    


    
        
            
                订单系统
            
            
            
                
                    订单系统是电商系统的核心模块之一，负责处理用户从下单到订单完成的整个流程。订单系统需要保证数据的一致性、处理高并发请求、支持复杂的业务逻辑，并提供良好的用户体验。
                
                
                订单系统功能
                
                
                    
                        
                            
                            购物车管理
                        
                        
                            商品添加、删除、修改数量
                            购物车商品库存检查
                            购物车数据持久化
                            跨设备购物车同步
                        
                    
                    
                    
                        
                            
                            订单生成
                        
                        
                            订单信息校验
                            订单价格计算
                            订单号生成
                            订单状态初始化
                        
                    
                    
                    
                        
                            
                            订单状态管理
                        
                        
                            待付款、待发货、待收货、已完成等状态
                            订单状态流转控制
                            订单超时处理
                            订单取消和退款
                        
                    
                    
                    
                        
                            
                            订单查询
                        
                        
                            订单列表查询
                            订单详情查看
                            订单搜索和筛选
                            订单统计分析
                        
                    
                
                
                订单流程设计
                
                
                    订单状态机
                    
                        
                            待付款
                            用户提交订单
                        
                        
                            待发货
                            用户完成支付
                        
                        
                            待收货
                            商家发货
                        
                        
                            已完成
                            用户确认收货
                        
                        
                            已取消
                            订单取消
                        
                    
                    
                    状态流转规则
                    
                        
                            
                                
                                    当前状态
                                    可流转状态
                                    触发条件
                                
                            
                            
                                
                                    待付款
                                    待发货, 已取消
                                    支付成功, 用户取消, 超时未支付
                                
                                
                                    待发货
                                    待收货, 已取消
                                    商家发货, 用户取消, 申请退款
                                
                                
                                    待收货
                                    已完成, 已取消
                                    用户确认收货, 用户拒收, 申请退款
                                
                            
                        
                    
                
                
                订单数据模型
                
                
                    核心数据表
                    
                        
                            
                                
                                    表名
                                    字段
                                    说明
                                
                            
                            
                                
                                    orders
                                    order_id, user_id, status, amount, create_time, update_time
                                    订单主表
                                
                                
                                    order_items
                                    item_id, order_id, product_id, quantity, price
                                    订单明细表
                                
                                
                                    shopping_cart
                                    cart_id, user_id, product_id, quantity, create_time
                                    购物车表
                                
                                
                                    order_status_log
                                    log_id, order_id, status, operator, create_time
                                    订单状态日志表
                                
                            
                        
                    
                    
                    订单号生成策略
                    
// 订单号生成示例
public class OrderNoGenerator {
    public static String generateOrderNo() {
        // 格式：业务标识 + 时间戳 + 随机数
        // 例如：DD202312011200010001
        StringBuilder sb = new StringBuilder();
        sb.append("DD"); // 业务标识
        sb.append(new SimpleDateFormat("yyyyMMddHHmmss").format(new Date()));
        sb.append(String.format("%04d", new Random().nextInt(10000)));
        return sb.toString();
    }
}
                
                
                高并发处理
                
                
                    
                        
                            
                            性能优化
                        
                        
                            订单数据分库分表
                            热点数据缓存
                            异步处理非核心流程
                            数据库连接池优化
                        
                    
                    
                    
                        
                            
                            限流策略
                        
                        
                            接口级限流
                            用户级限流
                            商品级限流
                            降级预案
                        
                    
                
                
                分布式事务
                
                
                    事务处理方案
                    
                        
                            TCC模式
                            
                                Try-Confirm-Cancel三阶段提交
                            
                            
                                Try阶段：检查并预留资源
                                Confirm阶段：确认执行
                                Cancel阶段：回滚操作
                            
                        
                        
                            可靠消息
                            
                                基于消息队列的最终一致性
                            
                            
                                本地事务执行
                                发送消息
                                消息消费
                                状态确认
                            
                        
                    
                    
                    订单创建事务示例
                    
// 基于可靠消息的订单创建
@Transactional
public Order createOrder(OrderRequest request) {
    // 1. 创建订单记录
    Order order = new Order();
    order.setOrderNo(generateOrderNo());
    order.setStatus(OrderStatus.PENDING_PAYMENT);
    orderMapper.insert(order);
    
    // 2. 扣减库存（发送消息）
    StockDeductMessage message = new StockDeductMessage();
    message.setOrderId(order.getId());
    message.setItems(request.getItems());
    messageProducer.send("stock_deduct_topic", message);
    
    // 3. 记录积分（发送消息）
    PointMessage pointMessage = new PointMessage();
    pointMessage.setUserId(request.getUserId());
    pointMessage.setOrderId(order.getId());
    pointMessage.setPoints(calculatePoints(request.getAmount()));
    messageProducer.send("point_topic", pointMessage);
    
    return order;
}
                
                
                异常处理
                
                
                    
                        
                            
                            常见异常场景
                        
                        
                            库存不足
                            价格变动
                            支付失败
                            网络超时
                            系统异常
                        
                    
                    
                    
                        
                            
                            补偿机制
                        
                        
                            定时任务检查未支付订单
                            库存回滚机制
                            支付状态同步
                            异常订单人工处理
                        
                    
                
                
                监控与运维
                
                
                    关键监控指标
                    
                        
                            性能指标
                            
                                订单创建平均耗时
                                订单查询QPS
                                数据库响应时间
                            
                        
                        
                            业务指标
                            
                                订单成功率
                                订单取消率
                                异常订单数
                            
                        
                        
                            系统指标
                            
                                缓存命中率
                                消息队列积压
                                线程池使用率
                            
                        
                    
                
                
                订单系统最佳实践
                
                    
                        合理的数据库设计和索引优化
                        完善的订单状态机设计
                        健壮的分布式事务处理方案
                        高效的缓存策略
                        全面的异常处理和补偿机制
                        详细的日志记录和监控告警
                        充分的压力测试和容量规划
