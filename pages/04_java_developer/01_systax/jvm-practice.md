JVM调优实战 - 架构师学习笔记
    
    


    
        
            
                JVM调优实战
            
            
            
                
                    JVM（Java Virtual Machine）是Java应用程序运行的核心环境。掌握JVM调优技能对于提升应用性能、解决内存问题和优化系统资源利用至关重要。
                
                
                
                    
                        JVM调优是Java应用性能优化的关键环节，通过合理的参数配置和监控可以显著提升应用的稳定性和性能。
                    
                
                
                JVM内存结构
                
                
                    
                        
                            堆内存(Heap)
                        
                        
                            存储对象实例和数组，是垃圾回收器管理的主要区域，分为新生代和老年代。
                        
                    
                    
                    
                        
                            栈内存(Stack)
                        
                        
                            存储局部变量、方法调用和部分结果，每个线程都有独立的栈空间。
                        
                    
                    
                    
                        
                            方法区(Metaspace)
                        
                        
                            存储类信息、常量、静态变量等，JDK 8后由永久代改为元空间。
                        
                    
                    
                    
                        
                            程序计数器
                        
                        
                            记录当前线程执行的字节码指令地址，是线程私有的内存区域。
                        
                    
                
                
                垃圾回收机制
                
                
                    
                        垃圾回收器类型
                        
# Serial GC - 单线程垃圾回收器
-XX:+UseSerialGC

# Parallel GC - 并行垃圾回收器（默认）
-XX:+UseParallelGC

# CMS GC - 并发标记清除垃圾回收器
-XX:+UseConcMarkSweepGC

# G1 GC - Garbage First垃圾回收器
-XX:+UseG1GC

# ZGC - 低延迟垃圾回收器（JDK 11+）
-XX:+UseZGC
                        
                            不同的垃圾回收器适用于不同的应用场景，需要根据应用特点选择合适的GC。
                        
                    
                    
                    
                        GC日志分析
                        
# 启用GC日志（JDK 9+）
-Xlog:gc*:gc.log:time,tags

# JDK 8及以下版本
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log

# GC日志示例分析
[GC (Allocation Failure) [PSYoungGen: 1024K->512K(2048K)] 2048K->1536K(4096K), 0.0012345 secs]
[Full GC (Ergonomics) [PSYoungGen: 512K->0K(2048K)] [ParOldGen: 1024K->800K(2048K)] 1536K->800K(4096K), [Metaspace: 2048K->2048K(10240K)], 0.0056789 secs]
                        
                            通过分析GC日志可以了解垃圾回收的频率、耗时和内存使用情况。
                        
                    
                
                
                JVM核心参数调优
                
                
                    
                        内存相关参数
                        
# 堆内存初始大小
-Xms2g

# 堆内存最大大小
-Xmx4g

# 新生代大小
-Xmn1g

# 设置元空间大小
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

# 直接内存大小
-XX:MaxDirectMemorySize=1g
                        
                            合理设置内存参数可以避免频繁的垃圾回收和内存溢出问题。
                        
                    
                    
                    
                        垃圾回收参数
                        
# G1垃圾回收器调优参数
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m
-XX:G1NewSizePercent=30
-XX:G1MaxNewSizePercent=40

# Parallel GC调优参数
-XX:+UseParallelGC
-XX:ParallelGCThreads=8
-XX:+UseAdaptiveSizePolicy

# CMS垃圾回收器调优参数
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=70
-XX:+UseCMSInitiatingOccupancyOnly
                        
                            根据应用特点和性能要求调整垃圾回收参数。
                        
                    
                
                
                性能监控与调优工具
                
                
                    
                        
                            jstat
                        
                        
# 查看GC统计信息
jstat -gc 

# 查看类加载信息
jstat -class 

# 查看编译统计信息
jstat -compiler 
                        
                            jstat是JDK自带的轻量级监控工具，可以实时查看JVM运行状态。
                        
                    
                    
                    
                        
                            jmap
                        
                        
# 查看堆内存使用情况
jmap -heap 

# 生成堆转储文件
jmap -dump:format=b,file=heap.hprof 

# 查看对象统计信息
jmap -histo 
                        
                            jmap用于生成堆内存快照，分析内存使用情况和内存泄漏问题。
                        
                    
                    
                    
                        
                            jstack
                        
                        
# 生成线程快照
jstack 

# 检测死锁
jstack -l 
                        
                            jstack用于生成线程快照，分析线程状态和死锁问题。
                        
                    
                    
                    
                        
                            VisualVM
                        
                        
                            图形化监控工具，集成了多种JDK命令行工具的功能，提供直观的性能分析界面。
                        
                    
                
                
                常见问题诊断与解决
                
                
                    
                        内存溢出(OOM)问题
                        
# 常见的OOM类型及解决方案

# 1. Java heap space
# 原因：堆内存不足
# 解决方案：增加堆内存(-Xmx)或优化代码减少对象创建

# 2. Metaspace
# 原因：元空间不足
# 解决方案：增加元空间大小(-XX:MaxMetaspaceSize)或检查类加载泄漏

# 3. GC overhead limit exceeded
# 原因：GC花费时间过长
# 解决方案：增加堆内存或优化内存使用

# 4. Direct buffer memory
# 原因：直接内存不足
# 解决方案：增加直接内存限制(-XX:MaxDirectMemorySize)
                        
                            针对不同类型的OOM问题采取相应的解决方案。
                        
                    
                    
                    
                        CPU使用率过高问题
                        
# 诊断步骤：
# 1. 使用top命令找到高CPU使用率的Java进程
top -p 

# 2. 查看该进程中高CPU使用率的线程
top -H -p 

# 3. 将线程ID转换为16进制
printf "%x\n" 

# 4. 使用jstack生成线程快照并查找对应线程
jstack  | grep -A 10 ""
                        
                            通过线程快照分析CPU使用率过高的原因。
                        
                    
                
                
                
                    JVM调优最佳实践
                    
                        根据应用特点选择合适的垃圾回收器
                        合理设置堆内存大小，避免过大或过小
                        监控GC日志，及时发现性能问题
                        定期进行性能测试和调优
                        使用专业的监控工具进行实时监控
                        建立完善的报警机制
