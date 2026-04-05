数据库分片 - 架构师学习笔记
    
    


    
        
            
                数据库分片
            
            
            
                
                    数据库分片是一种水平分割数据库的技术，将大型数据库分割成更小、更快、更容易管理的部分，这些部分称为数据分片。分片可以显著提高数据库的性能和可扩展性，是解决大数据量和高并发访问的有效手段。
                
                
                分片概述
                
                
                    
                        
                            
                            分片优势
                        
                        
                            提高系统性能和吞吐量
                            增强系统可扩展性
                            降低单个数据库的负载
                            提高系统可用性
                        
                    
                    
                    
                        
                            
                            分片挑战
                        
                        
                            增加系统复杂性
                            跨分片事务处理困难
                            数据分布不均问题
                            分片键选择困难
                        
                    
                
                
                分片策略
                
                
                    分片方式
                    
                        
                            水平分片
                            
                                按行分割数据，不同行存储在不同分片中
                            
                        
                        
                            垂直分片
                            
                                按列分割数据，不同列存储在不同分片中
                            
                        
                        
                            混合分片
                            
                                结合水平和垂直分片的优点
                            
                        
                    
                    
                    分片算法
                    
                        
                            
                                
                                    算法
                                    说明
                                    优点
                                    缺点
                                
                            
                            
                                
                                    哈希分片
                                    通过哈希函数计算分片位置
                                    数据分布均匀
                                    扩容困难
                                
                                
                                    范围分片
                                    根据字段值范围划分分片
                                    易于扩容
                                    数据分布可能不均
                                
                                
                                    列表分片
                                    根据字段值列表划分分片
                                    精确控制
                                    维护成本高
                                
                                
                                    复合分片
                                    结合多种分片策略
                                    灵活性高
                                    复杂度高
                                
                            
                        
                    
                
                
                分片键选择
                
                
                    
                        
                            
                            分片键特性
                        
                        
                            高基数：分片键值应具有较高的唯一性
                            均匀分布：分片键值应均匀分布
                            稳定性：分片键值应相对稳定，不易变更
                            查询相关性：分片键应与常用查询条件相关
                        
                    
                    
                    
                        
                            
                            避免的问题
                        
                        
                            避免使用单调递增字段作为分片键
                            避免使用热点数据集中的字段
                            避免使用频繁更新的字段
                            避免使用NULL值较多的字段
                        
                    
                
                
                分片实现方案
                
                
                    
                        
                            
                            应用层分片
                        
                        
                            在应用代码中实现分片逻辑，灵活性高但开发复杂。
                        
                        
// 分片路由示例
public class ShardingRouter {
    private int shardCount = 16;
    
    public int getShardIndex(long userId) {
        return (int) (userId % shardCount);
    }
    
    public DataSource getDataSource(long userId) {
        int shardIndex = getShardIndex(userId);
        return dataSourceMap.get("shard_" + shardIndex);
    }
}
                    
                    
                    
                        
                            
                            中间件分片
                        
                        
                            使用分片中间件实现分片，降低应用复杂度。
                        
                        
                            MyCat
                            ShardingSphere
                            Vitess
                        
                    
                
                
                ShardingSphere
                
                
                    分片配置示例
                    
# application.yml
spring:
  shardingsphere:
    datasource:
      names: ds0,ds1
      ds0:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/ds0
        username: root
        password: password
      ds1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/ds1
        username: root
        password: password
    
    rules:
      sharding:
        tables:
          t_order:
            actual-data-nodes: ds$->{0..1}.t_order_$->{0..1}
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: table-inline
            database-strategy:
              standard:
                sharding-column: user_id
                sharding-algorithm-name: database-inline
        sharding-algorithms:
          table-inline:
            type: INLINE
            props:
              algorithm-expression: t_order_$->{order_id % 2}
          database-inline:
            type: INLINE
            props:
              algorithm-expression: ds$->{user_id % 2}
                
                
                分布式事务
                
                
                    
                        
                            
                            事务类型
                        
                        
                            本地事务：单个数据库内的事务
                            分布式事务：跨多个数据库的事务
                            最终一致性：通过补偿机制实现一致性
                        
                    
                    
                    
                        
                            
                            解决方案
                        
                        
                            XA协议：两阶段提交，强一致性
                            Seata：AT模式，无侵入
                            TCC：Try-Confirm-Cancel模式
                            消息队列：通过消息实现最终一致性
                        
                    
                
                
                分片扩容
                
                
                    扩容策略
                    
                        
                            平滑扩容
                            
                                在不影响业务的情况下增加分片
                            
                            
                                一致性哈希算法
                                预分片策略
                                数据迁移工具
                            
                        
                        
                            数据迁移
                            
                                将现有数据重新分布到新的分片中
                            
                            
                                在线迁移
                                双写迁移
                                停机迁移
                            
                        
                    
                    
                    扩容步骤
                    
                        评估扩容需求和方案
                        准备新的数据库实例
                        修改分片规则配置
                        执行数据迁移
                        验证数据一致性
                        切换流量到新分片
                        清理旧分片数据
                    
                
                
                监控与运维
                
                
                    
                        
                            
                            监控指标
                        
                        
                            各分片的负载情况
                            查询响应时间
                            数据分布均匀性
                            连接池使用情况
                        
                    
                    
                    
                        
                            
                            运维工具
                        
                        
                            分片管理平台
                            数据迁移工具
                            SQL审计工具
                            性能分析工具
                        
                    
                
                
                分片最佳实践
                
                    
                        合理设计分片键，避免数据倾斜
                        预估业务增长，合理规划分片数量
                        选择合适的分片算法和中间件
                        实施完善的监控和告警机制
                        制定详细的数据迁移和扩容方案
                        处理好分布式事务和数据一致性
                        定期分析和优化分片策略
