\n\n    
    
    客户端卡顿问题定位案例 - 架构师学习笔记
    
    
    
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
        
            
                客户端卡顿问题定位案例
            
            Client Freeze Troubleshooting Case - 一次完整的客户端卡顿问题定位过程
            
            
                
                    
                    核心目标：通过实际案例展示如何定位客户端卡顿问题，从前端到后端，从接口到服务器资源，最终找到问题根源。
                
            

            
                问题现象
            

            
                
                    
                        客户端表现
                        
                            页面加载缓慢，操作响应延迟
                            某些功能点击后无响应或响应时间长
                            浏览器控制台无明显错误
                            网络请求显示某些接口响应时间超过5秒
                        
                    
                    
                        初步排查
                        
                            检查网络连接：网络正常，其他网站访问正常
                            检查浏览器：尝试不同浏览器，问题依然存在
                            检查前端代码：无明显错误，代码逻辑正常
                            检查网络请求：发现多个后端接口响应时间过长
                        
                    
                
            

            
                后端接口排查
            

            
                
                    
                        
                        接口响应时间分析
                    
                    
                        
                            使用浏览器开发者工具
                            
# 打开Network面板，查看接口响应时间
- 接口A：/api/user/list - 响应时间 6.2s
- 接口B：/api/order/detail - 响应时间 4.8s
- 接口C：/api/product/recommend - 响应时间 5.3s

# 观察请求状态
- 所有请求均为200 OK
- 响应数据大小正常
- 请求头和响应头无异常
                            
                        
                        
                            使用curl测试
                            
# 测试接口响应时间
curl -o /dev/null -s -w "%{time_total}" http://example.com/api/user/list
6.123

curl -o /dev/null -s -w "%{time_total}" http://example.com/api/order/detail
4.789

# 确认问题：后端接口确实响应缓慢
                            
                        
                    
                

                
                    
                        
                        服务器CPU监控
                    
                    
                        
                            检查服务器CPU使用情况
                            
# 登录服务器，查看CPU使用率
top

# 输出结果
top - 14:30:45 up 10 days,  2:15,  1 user,  load average: 8.23, 7.98, 7.65
Tasks: 123 total,   1 running, 122 sleeping,   0 stopped,   0 zombie
%Cpu(s): 95.2 us,  2.1 sy,  0.0 ni,  2.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :  16384.0 total,   1234.5 free,  12543.2 used,   2606.3 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   3215.6 avail Mem 

# 发现问题：CPU使用率高达95.2%，空闲仅2.7%
                            
                        
                        
                            查看占用CPU的进程
                            
# 查看进程CPU使用情况
top -b -n 1 | sort -k 9 -r | head -10

# 输出结果
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1234 java      20   0  8192m  2048m   128m S  92.5  12.5   2:34.56 java -jar app.jar
 5678 nginx     20   0   128m    32m    16m S   2.1   0.2   0:12.34 nginx: worker process
 9012 mysql     20   0  2048m   512m   256m S   1.8   3.1   1:23.45 mysqld

# 发现问题：Java进程占用了92.5%的CPU
                            
                        
                    
                
            

            
                Java进程排查
            

            
                
                    
                        
                        线程状态分析
                    
                    
                        
                            查看Java进程线程状态
                            
# 查看Java进程ID
ps aux | grep java

# 查看线程状态
top -H -p 1234

# 或者使用jstack
jstack 1234 > thread_dump.txt

# 分析线程转储
grep -A 10 "java.lang.Thread.State" thread_dump.txt | sort | uniq -c

# 输出结果
     10 RUNNABLE
      5 TIMED_WAITING (sleeping)
     23 TIMED_WAITING (parking)
     15 WAITING (parking)
      8 BLOCKED

# 发现问题：有8个线程处于BLOCKED状态
                            
                        
                        
                            分析阻塞线程
                            
# 查找阻塞线程
grep -B 5 -A 10 "BLOCKED" thread_dump.txt

# 输出结果示例
"http-nio-8080-exec-1" #23 daemon prio=5 os_prio=0 tid=0x00007f1a2c001000 nid=0x4d53 waiting for monitor entry [0x00007f19f45ff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.example.service.UserService.getUserList(UserService.java:45)
        - waiting to lock &lt;0x0000000080001234&gt; (a com.example.service.UserService)
        at sun.reflect.GeneratedMethodAccessor123.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:344)

# 发现问题：多个线程在等待UserService的锁
                            
                        
                    
                

                
                    
                        
                        数据库连接分析
                    
                    
                        
                            查看数据库连接池状态
                            
# 使用jstat查看JVM统计信息
jstat -gc 1234 1000 5

# 查看数据库连接池状态（假设使用HikariCP）
# 通过JMX或应用监控查看

# 检查数据库连接数
mysql -u root -p -e "show processlist;"

# 输出结果
+----+------+-----------+--------+---------+------+----------+------------------+
| Id | User | Host      | db     | Command | Time | State    | Info             |
+----+------+-----------+--------+---------+------+----------+------------------+
| 1  | app  | localhost | myapp  | Sleep   | 10   |          | NULL             |
| 2  | app  | localhost | myapp  | Sleep   | 15   |          | NULL             |
| 3  | app  | localhost | myapp  | Query   | 120  | executing| SELECT * FROM user |
| 4  | app  | localhost | myapp  | Query   | 115  | executing| SELECT * FROM user |
| 5  | app  | localhost | myapp  | Query   | 110  | executing| SELECT * FROM user |
+----+------+-----------+--------+---------+------+----------+------------------+

# 发现问题：多个数据库查询执行时间过长（超过100秒）
                            
                        
                        
                            分析SQL语句
                            
# 查看慢查询日志
cat /var/lib/mysql/slow-query.log

# 输出结果示例
# Time: 2026-04-01T14:30:00.000000Z
# User@Host: app[app] @ localhost []
# Query_time: 125.345231  Lock_time: 0.000123 Rows_sent: 10000  Rows_examined: 1000000
SET timestamp=1711957800;
SELECT * FROM user WHERE status = 'active' ORDER BY create_time DESC;

# 发现问题：SQL查询扫描了100万行数据，只返回1万行，且没有使用索引
                            
                        
                    
                

                
                    
                        
                        代码分析
                    
                    
                        
                            查看UserService代码
                            
// UserService.java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Synchronized
    public List&lt;User&gt; getUserList() {
        // 问题1：使用了@Synchronized注解，导致方法级别的同步锁
        // 问题2：没有使用分页，查询所有数据
        return userRepository.findByStatus("active");
    }
}

// UserRepository.java
public interface UserRepository extends JpaRepository&lt;User, Long&gt; {
    // 问题3：没有添加索引注解，且查询条件可能没有索引
    List&lt;User&gt; findByStatus(String status);
}
                            
                        
                        
                            数据库表结构分析
                            
# 查看user表结构
SHOW CREATE TABLE user;

# 输出结果
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `status` varchar(50) NOT NULL,
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  PRIMARY KEY (`id`),
  -- 缺少status字段的索引
  -- 缺少create_time字段的索引
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

# 发现问题：status和create_time字段都没有索引
                            
                        
                    
                
            

            
                问题定位总结
            

            
                
                    
                        问题根源
                        
                            
                                SQL查询问题：查询条件没有索引，导致全表扫描
                                同步锁问题：使用了@Synchronized注解，导致方法级别的同步锁，多个请求串行执行
                                分页问题：没有使用分页，一次性查询所有数据
                                数据库连接池问题：连接数不足，导致请求排队
                            
                        
                    
                    
                    
                        影响链路
                        
                            
                                客户端发起请求 → 后端接口
                                后端接口调用UserService.getUserList() → 获取同步锁
                                UserService调用userRepository.findByStatus() → 执行SQL查询
                                SQL查询没有索引 → 全表扫描，执行时间长
                                多个请求排队等待同步锁 → CPU使用率升高
                                数据库连接被长时间占用 → 连接池耗尽
                                后端接口响应缓慢 → 客户端卡顿
                            
                        
                    
                
            

            
                解决方案
            

            
                
                    
                        
                        数据库优化
                    
                    
                        
                            添加索引
                            
# 添加status字段索引
ALTER TABLE `user` ADD INDEX `idx_status` (`status`);

# 添加create_time字段索引
ALTER TABLE `user` ADD INDEX `idx_create_time` (`create_time`);
                            
                        
                    
                

                
                    
                        
                        代码优化
                    
                    
                        
                            修改UserService代码
                            
// 优化后的UserService.java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    // 移除@Synchronized注解，使用更细粒度的锁
    public List&lt;User&gt; getUserList(int page, int size) {
        // 添加分页参数
        Pageable pageable = PageRequest.of(page, size, Sort.by(Sort.Direction.DESC, "createTime"));
        return userRepository.findByStatus("active", pageable).getContent();
    }
}

// 优化后的UserRepository.java
public interface UserRepository extends JpaRepository&lt;User, Long&gt; {
    // 添加分页支持
    Page&lt;User&gt; findByStatus(String status, Pageable pageable);
}
                            
                        
                    
                

                
                    
                        
                        配置优化
                    
                    
                        
                            数据库连接池配置
                            
# application.yml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20  # 增加连接池大小
      minimum-idle: 10       # 最小空闲连接数
      idle-timeout: 30000    # 空闲连接超时时间
      max-lifetime: 1800000  # 连接最大生命周期
                            
                        
                    
                
            

            
                
                验证结果
            

            
                
                    
                        优化后效果
                        
                            接口响应时间：从6秒降至0.1秒以内
                            CPU使用率：从95%降至10%以下
                            数据库查询时间：从125秒降至0.01秒
                            客户端体验：卡顿现象完全消失
                        
                    
                    
                        验证方法
                        
                            使用浏览器开发者工具测试接口响应时间
                            使用top命令监控服务器CPU使用率
                            使用mysql slow query log监控SQL执行时间
                            进行压力测试，验证系统稳定性
                        
                    
                
            

            
                
                经验总结
            

            
                
                    
                    排查流程
                    从客户端到后端，从接口到服务器资源
                
                
                    
                    数据库优化
                    合理设计索引，优化SQL查询
                
                
                    
                    代码优化
                    避免过度同步，使用分页查询
                
                
                    
                    资源监控
                    实时监控服务器资源使用情况
                
                
                    
                    Java排查
                    使用jstack、jstat等工具分析Java进程
                
                
                    
                    持续优化
                    建立性能监控体系，定期优化
