\n\n    
    
    常见问题排查 - 架构师学习笔记
    
    
    
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
        
            
                常见问题排查
            
            Common Problems - 常见问题诊断与解决方案
            
            
                
                    
                    实用指南：收集开发和运维中最常见的问题，提供快速诊断步骤和解决方案。
                
            

            
                服务启动问题
            

            
                
                    
                        
                        端口被占用
                    
                    
                        
                            问题现象
                            服务启动失败，提示 "Address already in use" 或端口被占用
                        
                        
                            诊断步骤
                            
# 查看端口占用情况
netstat -tlnp | grep 8080
# 或者使用 lsof
lsof -i :8080
# 或者使用 ss
ss -tlnp | grep 8080
                            
                        
                        
                            解决方案
                            
# 1. 杀掉占用端口的进程
kill -9 &lt;pid&gt;
# 2. 或者修改服务配置，使用其他端口
server.port=8081
                            
                        
                    
                

                
                    
                        
                        数据库连接失败
                    
                    
                        
                            问题现象
                            服务启动时无法连接数据库，提示 "Communications link failure" 或 "Connection refused"
                        
                        
                            诊断步骤
                            
# 1. 测试数据库端口连通性
telnet localhost 3306
# 或者使用 nc
nc -zv localhost 3306
# 2. 检查数据库服务状态
systemctl status mysql
# 3. 检查数据库日志
tail -f /var/log/mysql/error.log
                            
                        
                        
                            常见原因
                            
                                数据库服务未启动
                                防火墙/安全组未开放端口
                                数据库配置错误（主机、端口、用户名、密码）
                                连接数超过数据库最大连接数
                            
                        
                    
                
            

            
                性能问题
            

            
                
                    
                        
                        CPU使用率过高
                    
                    
                        
                            诊断工具
                            
# 1. 使用 top 查看整体情况
top
# 2. 使用 htop（更友好的界面）
htop
# 3. 查看Java进程线程
top -Hp &lt;pid&gt;
# 4. 使用 jstack 查看线程堆栈
jstack &lt;pid&gt; &gt; thread_dump.txt
# 5. 使用 Arthas
java -jar arthas-boot.jar
                            
                        
                        
                            常见原因
                            
                                死循环或无限递归
                                频繁的GC（垃圾回收）
                                大量线程竞争锁
                                CPU密集型计算
                            
                        
                    
                

                
                    
                        
                        内存泄漏/溢出
                    
                    
                        
                            诊断工具
                            
# 1. 查看堆内存使用情况
jmap -heap &lt;pid&gt;
# 2. 导出堆内存dump
jmap -dump:format=b,file=heap_dump.hprof &lt;pid&gt;
# 3. 使用 jstat 监控GC
jstat -gcutil &lt;pid&gt; 1000
# 4. 使用 MAT 分析堆dump
# Memory Analyzer Tool
                            
                        
                        
                            常见原因
                            
                                静态集合类未清理
                                监听器未注销
                                连接池资源未释放
                                ThreadLocal变量未清理
                            
                        
                    
                
            

            
                网络问题
            

            
                
                    
                        
                        请求超时
                    
                    
                        
                            诊断步骤
                            
# 1. 使用 curl 测试接口
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:8080/api
# curl-format.txt 内容：
# time_namelookup:  %{time_namelookup}\n
# time_connect:  %{time_connect}\n
# time_appconnect:  %{time_appconnect}\n
# time_pretransfer:  %{time_pretransfer}\n
# time_redirect:  %{time_redirect}\n
# time_starttransfer:  %{time_starttransfer}\n
# ----------\n
# time_total:  %{time_total}\n

# 2. 使用 tcpdump 抓包分析
tcpdump -i any -w network.pcap port 8080
# 然后用 Wireshark 打开分析
                            
                        
                        
                            常见原因
                            
                                数据库查询慢
                                网络带宽不足
                                下游服务响应慢
                                GC STW时间过长
                            
                        
                    
                

                
                    
                        
                        连接被重置 (Connection Reset)
                    
                    
                        
                            诊断步骤
                            
# 1. 检查防火墙状态
systemctl status firewalld
iptables -L -n
# 2. 检查 TCP 参数
sysctl net.ipv4.tcp_fin_timeout
sysctl net.ipv4.tcp_keepalive_time
# 3. 检查网络连接数
netstat -an | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
                            
                        
                        
                            常见原因
                            
                                防火墙主动断开连接
                                服务端主动关闭连接
                                网络中间设备（负载均衡）超时
                                客户端异常退出
                            
                        
                    
                
            

            
                Redis问题
            

            
                
                    
                        
                        Redis连接超时
                    
                    
                        
                            诊断步骤
                            
# 1. 测试Redis连通性
redis-cli -h localhost -p 6379 ping
# 2. 查看Redis信息
redis-cli info
# 3. 查看慢查询
redis-cli slowlog get 10
# 4. 监控Redis命令
redis-cli monitor
                            
                        
                        
                            常见原因
                            
                                Redis服务CPU/内存高负载
                                大key操作导致阻塞
                                连接数超过maxclients
                                网络延迟高
                            
                        
                    
                

                
                    
                        
                        内存溢出 (OOM)
                    
                    
                        
                            诊断步骤
                            
# 1. 查看内存使用
redis-cli info memory
# 2. 查找大key
redis-cli --bigkeys
# 3. 查看内存淘汰策略
redis-cli config get maxmemory-policy
# 4. 分析key分布
redis-cli --scan | head -100
                            
                        
                        
                            解决方案
                            
                                设置合理的maxmemory和淘汰策略
                                设置key过期时间
                                拆分大key
                                升级Redis内存或使用集群
                            
                        
                    
                
            

            
                排查检查清单
            
            
            
                
                    
                    服务健康
                    进程、端口、日志
                
                
                    
                    数据库
                    连接、慢查询、锁
                
                
                    
                    内存
                    堆内存、GC、泄漏
                
                
                    
                    CPU
                    使用率、线程、死锁
                
                
                    
                    网络
                    连通性、延迟、丢包
                
                
                    
                    配置
                    参数、依赖、版本
