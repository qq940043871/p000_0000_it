# JVM原理 - 架构师学习笔记

## JVM原理

Java虚拟机（JVM）是Java平台的核心组件，它负责执行Java字节码，提供内存管理、垃圾回收、即时编译等核心功能。深入理解JVM原理对于Java性能调优和故障排查至关重要。

### JVM架构

#### JVM组成

| 组件 | 功能 |
|------|------|
| 类加载器（Class Loader） | 负责加载.class文件到内存 |
| 运行时数据区（Runtime Data Area） | 管理JVM运行时的内存区域 |
| 执行引擎（Execution Engine） | 执行字节码指令 |
| 本地方法接口（JNI） | 与本地方法库交互 |
| 本地方法库（Native Method Libraries） | 本地方法的实现 |

### 运行时数据区

#### 方法区（Method Area）

- 存储类信息、常量、静态变量等
- 被所有线程共享
- 在JDK 8中被元空间（Metaspace）替代

#### 堆（Heap）

- 存储对象实例和数组
- 被所有线程共享
- 是垃圾回收的主要区域

#### 虚拟机栈（VM Stack）

- 存储局部变量表、操作数栈等
- 每个线程私有
- StackOverflowError异常发生地

#### 本地方法栈（Native Method Stack）

- 为本地方法服务
- 每个线程私有
- 与虚拟机栈类似但为Native方法服务

#### 程序计数器（Program Counter Register）

- 记录当前线程执行的字节码指令地址
- 每个线程私有
- 是唯一不会出现OutOfMemoryError的区域

### 垃圾回收机制

#### 垃圾回收算法

- 标记-清除算法：标记所有需要回收的对象，统一回收
- 复制算法：将存活对象复制到另一块内存区域
- 标记-整理算法：标记后将存活对象向一端移动
- 分代收集算法：根据对象存活周期将内存分为几块

#### 垃圾收集器

- Serial收集器：单线程收集器
- ParNew收集器：Serial收集器的多线程版本
- Parallel Scavenge收集器：关注吞吐量的收集器
- CMS收集器：以最短回收停顿时间为目标
- G1收集器：面向服务端应用的收集器

### 类加载机制

#### 双亲委派模型

类加载器之间的层次关系，保证Java程序的稳定运行。

| 类加载器 | 加载类的范围 |
|---------|------------|
| 启动类加载器（Bootstrap ClassLoader） | 加载%JAVA_HOME%/lib目录下的类库 |
| 扩展类加载器（Extension ClassLoader） | 加载%JAVA_HOME%/lib/ext目录下的类库 |
| 应用程序类加载器（Application ClassLoader） | 加载用户类路径上指定的类库 |

#### 工作原理

- 类加载器收到类加载请求时，不会自己去加载，而是委派给父类加载器
- 父类加载器无法加载时，才会尝试自己加载
- 保证核心类库的安全性和一致性

### JVM性能调优

#### 内存调优参数

- -Xms：设置堆的初始大小
- -Xmx：设置堆的最大大小
- -Xmn：设置新生代大小
- -XX:NewRatio：设置老年代与新生代的比例
- -XX:SurvivorRatio：设置Eden区与Survivor区的比例

#### 垃圾收集器调优

- -XX:+UseSerialGC：使用Serial+Serial Old收集器
- -XX:+UseParNewGC：使用ParNew+Serial Old收集器
- -XX:+UseParallelGC：使用Parallel Scavenge+Parallel Old收集器
- -XX:+UseConcMarkSweepGC：使用ParNew+CMS+Serial Old收集器
- -XX:+UseG1GC：使用G1收集器

### 常见问题诊断

#### 常见JVM异常

- OutOfMemoryError：内存不足，需要调整内存参数或优化代码
- StackOverflowError：栈溢出，通常是递归调用过深
- ClassNotFoundException：类未找到，检查类路径配置
- NoClassDefFoundError：编译时能找到类，运行时找不到

#### 诊断工具

- jps：查看Java进程
- jstat：监控JVM统计信息
- jmap：生成堆转储快照
- jstack：生成线程快照
- VisualVM：可视化监控工具
