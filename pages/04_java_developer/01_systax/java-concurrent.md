Java并发编程 - 架构师学习笔记
    
    


    
        
            
                Java并发编程
            
            
            
                
                    Java并发编程是Java核心技术之一，它允许程序同时执行多个任务，提高系统性能和资源利用率。在多核处理器普及的今天，掌握并发编程技术对于构建高性能应用至关重要。
                
                
                线程基础
                
                
                    
                        
                            
                            创建线程的方式
                        
                        
                            继承Thread类：创建Thread子类并重写run方法
                            实现Runnable接口：实现Runnable接口的run方法
                            实现Callable接口：实现Callable接口的call方法，支持返回值
                            使用线程池：通过ExecutorService管理线程
                        
                    
                    
                    
                        
                            
                            线程状态
                        
                        
                            NEW：新建状态，线程被创建但未启动
                            RUNNABLE：就绪状态，等待CPU调度
                            BLOCKED：阻塞状态，等待监视器锁
                            WAITING：等待状态，无限期等待其他线程
                            TIMED_WAITING：超时等待状态，等待指定时间
                            TERMINATED：终止状态，线程执行完毕
                        
                    
                
                
                同步机制
                
                
                    synchronized关键字
                    
                        synchronized是Java中最基本的同步机制，它可以修饰实例方法、静态方法和代码块，确保同一时刻只有一个线程执行被synchronized修饰的代码。
                    
                    
                    
// synchronized修饰实例方法
public synchronized void method1() {
    // 同步代码块
}

// synchronized修饰静态方法
public static synchronized void method2() {
    // 同步代码块
}

// synchronized修饰代码块
public void method3() {
    synchronized(this) {
        // 同步代码块
    }
}
                
                
                
                    Lock接口
                    
                        Lock接口提供了比synchronized更灵活的锁定操作，支持公平锁、可中断锁、超时锁等特性。
                    
                    
                    
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
    
    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
                
                
                并发工具类
                
                
                    
                        
                            
                            CountDownLatch
                        
                        
                            允许一个或多个线程等待其他线程完成操作。
                        
                        
CountDownLatch latch = new CountDownLatch(3);
// 在其他线程中调用latch.countDown()
latch.await(); // 等待计数器归零

                    
                    
                    
                        
                            
                            CyclicBarrier
                        
                        
                            让一组线程等待彼此到达一个共同的屏障点。
                        
                        
CyclicBarrier barrier = new CyclicBarrier(3);
// 在线程中调用barrier.await()

                    
                    
                    
                        
                            
                            Semaphore
                        
                        
                            控制同时访问特定资源的线程数量。
                        
                        
Semaphore semaphore = new Semaphore(2);
semaphore.acquire(); // 获取许可
// 访问资源
semaphore.release(); // 释放许可

                    
                    
                    
                        
                            
                            BlockingQueue
                        
                        
                            支持阻塞的插入和移除操作的队列。
                        
                        
BlockingQueue queue = new ArrayBlockingQueue(10);
queue.put("item"); // 阻塞插入
String item = queue.take(); // 阻塞获取

                    
                
                
                线程池
                
                
                    ExecutorService
                    
                        线程池是管理线程的工具，可以有效地控制线程数量，减少创建和销毁线程的开销。
                    
                    
                    
// 创建固定大小的线程池
ExecutorService executor = Executors.newFixedThreadPool(5);

// 提交任务
executor.submit(() -> {
    System.out.println("Task executed by " + Thread.currentThread().getName());
});

// 关闭线程池
executor.shutdown();
                    
                    线程池类型
                    
                        newFixedThreadPool：固定大小线程池
                        newCachedThreadPool：缓存线程池，根据需要创建新线程
                        newSingleThreadExecutor：单线程线程池
                        newScheduledThreadPool：支持定时及周期性任务执行的线程池
                    
                
                
                并发集合
                
                
                    
                        
                            
                                集合类
                                特点
                                适用场景
                            
                        
                        
                            
                                ConcurrentHashMap
                                线程安全的HashMap
                                高并发读写场景
                            
                            
                                CopyOnWriteArrayList
                                写时复制的List
                                读多写少场景
                            
                            
                                BlockingQueue
                                支持阻塞操作的队列
                                生产者消费者模式
                            
                            
                                ConcurrentLinkedQueue
                                非阻塞的线程安全队列
                                高并发队列操作
                            
                        
                    
                
                
                最佳实践
                
                    
                        避免使用过时的Thread.stop()、suspend()和resume()方法
                        合理使用volatile关键字保证变量的可见性
                        避免在同步块中执行耗时操作
                        正确处理线程中断，不要忽略InterruptedException
                        使用线程池管理线程，避免频繁创建和销毁线程
                        选择合适的并发工具类解决特定问题
                        注意死锁问题，避免嵌套锁
