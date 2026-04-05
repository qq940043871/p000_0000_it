\n\n    
    
    系统压力测试 - 架构师学习笔记
    
    
    
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
        
            
                系统压力测试
            
            System Stress Testing - 正确进行压力测试并提供专业的压测报告
            
            
                
                    
                    核心目标：掌握系统压力测试方法，生成专业的压测报告，为系统优化和容量规划提供依据。
                
            

            
                压测工具
            
            
            
                
                    
                    JMeter
                    Apache JMeter
                
                
                    
                    Gatling
                    基于Scala
                
                
                    
                    Locust
                    基于Python
                
                
                    
                    ab
                    Apache Bench
                
            

            
                压测流程
            

            
                
                    
                    
                        
                            1
                            
                                需求分析
                                明确测试目标、场景、指标和预期结果
                            
                        
                        
                            2
                            
                                测试准备
                                环境准备、数据准备、工具配置
                            
                        
                        
                            3
                            
                                执行测试
                                执行不同场景的压力测试，收集数据
                            
                        
                        
                            4
                            
                                数据分析
                                分析测试数据，找出瓶颈和问题
                            
                        
                        
                            5
                            
                                报告生成
                                生成专业的压测报告，提供优化建议
                            
                        
                    
                
            

            
                压测工具使用
            

            
                
                    
                        
                        Apache JMeter
                    
                    
                        
                            基本配置
                            
# 1. 下载JMeter
# 2. 启动JMeter
./jmeter.sh
# 3. 创建测试计划
# 4. 添加线程组
# 5. 添加HTTP请求
# 6. 添加监听器
                            
                        
                        
                            常用命令
                            
# 命令行模式运行
./jmeter -n -t test.jmx -l results.jtl -e -o report

# 配置参数
-n: 非GUI模式
-t: 测试计划文件
-l: 结果文件
-e: 生成报告
-o: 报告输出目录
                            
                        
                        
                            最佳实践
                            
                                使用非GUI模式进行大规模测试
                                合理设置线程组参数
                                添加适当的定时器
                                使用CSV数据文件参数化
                                监控服务器资源使用
                            
                        
                    
                

                
                    
                        
                        Gatling
                    
                    
                        
                            基本配置
                            
# 1. 下载Gatling
# 2. 编写测试脚本
# 3. 运行测试
./bin/gatling.sh

# 4. 查看报告
# 报告位置: results/
                            
                        
                        
                            测试脚本示例
                            
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class BasicSimulation extends Simulation {
  val httpProtocol = http
    .baseUrl("http://localhost:8080")
    .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
    .doNotTrackHeader("1")

  val scn = scenario("Basic Scenario")
    .exec(http("Home Page")
      .get("/"))

  setUp(
    scn.inject(
      rampUsers(100).during(5.seconds),
      constantUsersPerSec(10).during(10.seconds)
    )
  ).protocols(httpProtocol)
}
                            
                        
                        
                            优势
                            
                                基于Scala，性能更好
                                内置监控和报告
                                支持DSL编写测试
                                更高的并发支持
                            
                        
                    
                

                
                    
                        
                        Locust
                    
                    
                        
                            基本配置
                            
# 1. 安装Locust
pip install locust

# 2. 编写测试脚本
# 3. 启动Locust
locust -f locustfile.py

# 4. 访问Web界面
# http://localhost:8089
                            
                        
                        
                            测试脚本示例
                            
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)

    @task
    def index_page(self):
        self.client.get("/")

    @task(3)
    def view_item(self):
        for item_id in range(10):
            self.client.get(f"/item?id={item_id}")
            self.wait()

    def on_start(self):
        self.client.post("/login", {
            "username": "test",
            "password": "test"
        })
                            
                        
                        
                            优势
                            
                                基于Python，易于编写
                                Web界面实时监控
                                支持分布式测试
                                灵活的测试场景定义
                            
                        
                    
                

                
                    
                        
                        Apache Bench (ab)
                    
                    
                        
                            基本使用
                            
# 安装
sudo apt install apache2-utils

# 基本测试
ab -n 1000 -c 100 http://localhost:8080/

# 参数说明
-n: 总请求数
-c: 并发数
-t: 测试时长
-k: 启用Keep-Alive
                            
                        
                        
                            输出示例
                            
Server Software:        nginx/1.18.0
Server Hostname:        localhost
Server Port:            8080

Document Path:          /
Document Length:        123 bytes

Concurrency Level:      100
Time taken for tests:   2.345 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      234567 bytes
HTML transferred:       123000 bytes
Requests per second:    426.47 [#/sec] (mean)
Time per request:       234.49 [ms] (mean)
Time per request:       2.34 [ms] (mean, across all concurrent requests)
Transfer rate:          97.89 [Kbytes/sec] received

Connection Times (ms):
              min  mean[+/-sd] median   max
Connect:        0    1   0.5      1       5
Processing:     1  232  12.3    230     250
Waiting:        1  230  12.0    228     248
Total:          1  233  12.3    231     255

Percentage of the requests served within a certain time (ms):
  50%    231
  66%    235
  75%    238
  80%    240
  90%    245
  95%    248
  98%    252
  99%    254
 100%    255 (longest request)
                            
                        
                        
                            适用场景
                            
                                快速测试简单接口
                                验证基本性能指标
                                比较不同配置的性能
                                不适合复杂场景
                            
                        
                    
                
            

            
                压测报告模板
            

            
                
                    
                        1. 压测概述
                        
                            
                                测试目的：验证系统在不同负载下的性能表现
                                测试环境：生产环境/测试环境
                                测试时间：YYYY-MM-DD HH:MM-HH:MM
                                测试工具：JMeter/Gatling/Locust
                                测试场景：正常流量、峰值流量、极限压力
                            
                        
                    
                    
                    
                        2. 测试环境
                        
                            服务器配置
                            
                                
                                    
                                        项目
                                        配置
                                    
                                
                                
                                    
                                        CPU
                                        8核16线程
                                    
                                    
                                        内存
                                        16GB
                                    
                                    
                                        磁盘
                                        SSD 200GB
                                    
                                    
                                        网络
                                        1Gbps
                                    
                                
                            
                        
                    

                    
                        3. 测试结果
                        
                            主要指标
                            
                                
                                    
                                        场景
                                        并发数
                                        QPS
                                        响应时间
                                        错误率
                                    
                                
                                
                                    
                                        正常流量
                                        100
                                        500
                                        200ms
                                        0%
                                    
                                    
                                        峰值流量
                                        500
                                        2000
                                        500ms
                                        0.5%
                                    
                                    
                                        极限压力
                                        1000
                                        3000
                                        1000ms
                                        5%
                                    
                                
                            
                        
                    

                    
                        4. 资源使用
                        
                            服务器资源
                            
                                
                                    
                                        资源
                                        正常流量
                                        峰值流量
                                        极限压力
                                    
                                
                                
                                    
                                        CPU
                                        30%
                                        70%
                                        95%
                                    
                                    
                                        内存
                                        40%
                                        60%
                                        80%
                                    
                                    
                                        磁盘IO
                                        20%
                                        40%
                                        60%
                                    
                                    
                                        网络
                                        10%
                                        30%
                                        50%
                                    
                                
                            
                        
                    

                    
                        5. 分析与建议
                        
                            问题分析
                            
                                瓶颈识别：在并发1000时，CPU使用率达到95%，成为主要瓶颈
                                响应时间：超过800ms时用户体验开始下降
                                错误率：超过5%的错误率会影响系统稳定性
                            
                            
                            优化建议
                            
                                硬件升级：增加CPU核心数，升级到16核
                                代码优化：优化数据库查询，减少CPU密集型操作
                                缓存策略：增加缓存命中率，减少数据库访问
                                负载均衡：增加服务器节点，实现水平扩展
                                限流策略：实施合理的限流机制，保护系统
                            
                        
                    

                    
                        6. 结论
                        
                            
                                系统当前配置能够支撑的最大并发约为500，QPS约为2000，响应时间保持在500ms以内，错误率低于1%。
                            
                            
                                建议在生产环境部署时，配置至少8核16GB内存的服务器，并实施合理的负载均衡策略，以应对峰值流量。
                            
                        
                    
                
            

            
                压测最佳实践
            
            
            
                
                    
                    测试准备
                    明确目标、准备环境、数据
                
                
                    
                    渐进加压
                    从低并发逐渐增加
                
                
                    
                    监控资源
                    实时监控服务器资源
                
                
                    
                    数据准备
                    使用真实数据进行测试
                
                
                    
                    报告专业
                    生成详细的压测报告
                
                
                    
                    工具选择
                    根据场景选择合适工具
