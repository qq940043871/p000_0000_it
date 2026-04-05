库存系统 - 架构师学习笔记
    
    


    
        
            
                库存系统
            
            
            
                
                    库存系统是电商系统的重要组成部分，负责管理商品的库存数量、库存状态、库存预警等核心功能。库存系统的稳定性和准确性直接影响到用户的购物体验和企业的运营效率。
                
                
                库存系统功能
                
                
                    
                        
                            
                            库存管理
                        
                        
                            商品库存数量管理
                            库存状态跟踪
                            库存分类管理
                            库存预警设置
                        
                    
                    
                    
                        
                            
                            库存操作
                        
                        
                            入库操作
                            出库操作
                            库存调整
                            库存盘点
                        
                    
                    
                    
                        
                            
                            库存预警
                        
                        
                            低库存预警
                            超卖预警
                            滞销商品预警
                            库存异常预警
                        
                    
                    
                    
                        
                            
                            库存同步
                        
                        
                            多仓库库存同步
                            分布式库存同步
                            实时库存更新
                            库存数据一致性
                        
                    
                
                
                库存模型设计
                
                
                    库存类型
                    
                        
                            现货库存
                            
                                实际存储在仓库中的商品数量
                            
                        
                        
                            虚拟库存
                            
                                可销售的库存数量，包含预售等
                            
                        
                        
                            锁定库存
                            
                                已被订单占用但未支付的库存
                            
                        
                    
                    
                    核心数据表
                    
                        
                            
                                
                                    表名
                                    字段
                                    说明
                                
                            
                            
                                
                                    inventory
                                    sku_id, warehouse_id, quantity, locked_quantity, available_quantity
                                    库存主表
                                
                                
                                    inventory_log
                                    log_id, sku_id, operation_type, quantity, operator, create_time
                                    库存操作日志
                                
                                
                                    warehouse
                                    warehouse_id, name, address, status
                                    仓库信息表
                                
                            
                        
                    
                
                
                库存扣减策略
                
                
                    
                        
                            
                            实时扣减
                        
                        
                            用户下单时立即扣减库存，保证库存准确性。
                        
                        
                            优点：库存准确，避免超卖
                            缺点：影响下单性能，可能被恶意占用
                        
                    
                    
                    
                        
                            
                            延迟扣减
                        
                        
                            用户支付成功后再扣减库存，提高下单成功率。
                        
                        
                            优点：提高下单成功率
                            缺点：可能超卖，需要补偿机制
                        
                    
                
                
                高并发处理
                
                
                    并发控制方案
                    
                        
                            数据库锁
                            
                                使用行级锁保证数据一致性
                            
                            
SELECT * FROM inventory 
WHERE sku_id = ? 
FOR UPDATE;
                        
                        
                            分布式锁
                            
                                使用Redis等实现分布式锁
                            
                            
RedisLock lock = new RedisLock("inventory:" + skuId);
try {
    if (lock.tryLock()) {
        // 扣减库存
    }
} finally {
    lock.unlock();
}
                        
                        
                            乐观锁
                            
                                使用版本号控制并发更新
                            
                            
UPDATE inventory 
SET quantity = quantity - ?, version = version + 1 
WHERE sku_id = ? AND version = ?;
                        
                    
                    
                    库存扣减示例
                    
// 基于Redis的库存扣减
@Service
public class InventoryService {
    
    public boolean deductInventory(String skuId, int quantity) {
        String key = "inventory:" + skuId;
        String lockKey = "inventory_lock:" + skuId;
        
        // 获取分布式锁
        if (!redisTemplate.opsForValue().setIfAbsent(lockKey, "1", 10, TimeUnit.SECONDS)) {
            throw new InventoryException("获取库存锁失败");
        }
        
        try {
            // 获取当前库存
            Integer currentStock = (Integer) redisTemplate.opsForValue().get(key);
            if (currentStock == null || currentStock 
                
                
                分布式库存
                
                
                    
                        
                            
                            多仓库管理
                        
                        
                            仓库区域划分
                            库存区域分配
                            跨仓库调拨
                            就近发货策略
                        
                    
                    
                    
                        
                            
                            数据同步
                        
                        
                            主从同步
                            消息队列同步
                            定时任务同步
                            增量同步
                        
                    
                
                
                库存预警机制
                
                
                    预警类型
                    
                        
                            低库存预警
                            
                                当库存低于设定阈值时触发预警
                            
                            
                                邮件通知
                                短信提醒
                                系统消息
                            
                        
                        
                            超卖预警
                            
                                当库存出现负数时触发预警
                            
                            
                                自动补货
                                订单拦截
                                人工处理
                            
                        
                    
                    
                    预警配置示例
                    
// 库存预警配置
@Component
public class InventoryAlertService {
    
    @Scheduled(fixedRate = 300000) // 每5分钟检查一次
    public void checkInventoryAlert() {
        // 查询低库存商品
        List lowStockItems = inventoryMapper
            .selectLowStockItems(threshold);
        
        if (!lowStockItems.isEmpty()) {
            // 发送预警通知
            notificationService.sendLowStockAlert(lowStockItems);
        }
        
        // 查询超卖商品
        List oversoldItems = inventoryMapper
            .selectOversoldItems();
        
        if (!oversoldItems.isEmpty()) {
            // 发送超卖预警
            notificationService.sendOversoldAlert(oversoldItems);
        }
    }
}
                
                
                异常处理
                
                
                    
                        
                            
                            常见异常场景
                        
                        
                            库存不足
                            并发超卖
                            网络超时
                            系统异常
                        
                    
                    
                    
                        
                            
                            补偿机制
                        
                        
                            库存回滚
                            定时任务补偿
                            人工干预处理
                            数据一致性校验
                        
                    
                
                
                监控与运维
                
                
                    关键监控指标
                    
                        
                            性能指标
                            
                                库存查询响应时间
                                库存扣减成功率
                                并发处理能力
                            
                        
                        
                            业务指标
                            
                                库存准确率
                                超卖发生次数
                                预警触发次数
                            
                        
                        
                            系统指标
                            
                                缓存命中率
                                数据库连接数
                                锁等待时间
                            
                        
                    
                
                
                库存系统最佳实践
                
                    
                        合理的库存模型设计
                        高效的并发控制机制
                        完善的预警和补偿机制
                        准确的数据同步方案
                        全面的监控和告警体系
                        充分的压力测试和容量规划
                        详细的日志记录和追踪
