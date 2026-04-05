Java集合框架 - 架构师学习笔记
    
    


    
        
            
                Java集合框架
            
            
            
                
                    Java集合框架是Java编程语言中用来表示和操作集合的统一架构。所有的集合框架都包含在java.util包中，它提供了许多接口和实现类，用于存储和操作对象组。
                
                
                集合框架体系结构
                
                
                    核心接口
                    
                        
                            
                                
                                    接口
                                    描述
                                    主要实现类
                                
                            
                            
                                
                                    Collection
                                    集合层次结构的根接口
                                    List, Set, Queue
                                
                                
                                    List
                                    有序集合，允许重复元素
                                    ArrayList, LinkedList, Vector
                                
                                
                                    Set
                                    不允许重复元素的集合
                                    HashSet, TreeSet, LinkedHashSet
                                
                                
                                    Queue
                                    队列，通常以FIFO方式操作元素
                                    LinkedList, PriorityQueue
                                
                                
                                    Map
                                    键值对映射，键唯一
                                    HashMap, TreeMap, LinkedHashMap
                                
                            
                        
                    
                
                
                常用集合类对比
                
                
                    
                        
                            
                            List实现类
                        
                        
                            
                                
                                    
                                        实现类
                                        底层结构
                                        特点
                                    
                                
                                
                                    
                                        ArrayList
                                        动态数组
                                        查询快，增删慢
                                    
                                    
                                        LinkedList
                                        双向链表
                                        增删快，查询慢
                                    
                                    
                                        Vector
                                        动态数组
                                        线程安全
                                    
                                
                            
                        
                    
                    
                    
                        
                            
                            Set实现类
                        
                        
                            
                                
                                    
                                        实现类
                                        底层结构
                                        特点
                                    
                                
                                
                                    
                                        HashSet
                                        哈希表
                                        无序，查询快
                                    
                                    
                                        LinkedHashSet
                                        哈希表+链表
                                        有序，查询快
                                    
                                    
                                        TreeSet
                                        红黑树
                                        有序，排序存储
                                    
                                
                            
                        
                    
                
                
                Map实现类对比
                
                
                    
                        
                            
                                
                                    实现类
                                    底层结构
                                    特点
                                    适用场景
                                
                            
                            
                                
                                    HashMap
                                    哈希表
                                    无序，查询快
                                    一般场景
                                
                                
                                    LinkedHashMap
                                    哈希表+链表
                                    有序，查询快
                                    需要保持插入顺序
                                
                                
                                    TreeMap
                                    红黑树
                                    有序，排序存储
                                    需要排序的场景
                                
                                
                                    Hashtable
                                    哈希表
                                    线程安全
                                    线程安全场景
                                
                            
                        
                    
                
                
                使用示例
                
                
// ArrayList示例
List arrayList = new ArrayList();
arrayList.add("Apple");
arrayList.add("Banana");
arrayList.add("Orange");

// LinkedList示例
List linkedList = new LinkedList();
linkedList.add("Dog");
linkedList.add("Cat");
linkedList.add("Bird");

// HashSet示例
Set hashSet = new HashSet();
hashSet.add("Red");
hashSet.add("Green");
hashSet.add("Blue");

// HashMap示例
Map hashMap = new HashMap();
hashMap.put("Apple", 5);
hashMap.put("Banana", 3);
hashMap.put("Orange", 8);

// 遍历示例
System.out.println("ArrayList:");
for (String fruit : arrayList) {
    System.out.println(fruit);
}

System.out.println("\nHashMap:");
for (Map.Entry entry : hashMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
                
                性能优化建议
                
                    
                        根据使用场景选择合适的集合类
                        预估集合大小时设置初始容量，避免频繁扩容
                        注意集合的线程安全性，在并发环境下使用线程安全的集合类
                        合理使用迭代器遍历集合
                        避免在遍历集合时修改集合结构
                        使用泛型避免类型转换错误
