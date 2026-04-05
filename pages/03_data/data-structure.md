# 数据结构 - 架构师学习笔记

## 数据结构概述

数据结构是计算机存储、组织数据的方式，是指相互之间存在一种或多种特定关系的数据元素的集合。良好的数据结构设计可以提高算法的效率，是计算机科学的基础。

## 常见数据结构

### 线性结构

#### 数组（Array）

- **特点**：连续存储，随机访问，固定大小
- **时间复杂度**：访问 O(1)，插入/删除 O(n)
- **适用场景**：需要频繁访问元素，已知数据规模

**实现示例**：

```java
// 一维数组
int[] arr = new int[10];
arr[0] = 1;
arr[1] = 2;

// 二维数组
int[][] matrix = new int[3][3];
matrix[0][0] = 1;
```

#### 链表（Linked List）

- **特点**：非连续存储，通过指针连接，动态大小
- **时间复杂度**：访问 O(n)，插入/删除 O(1)
- **适用场景**：频繁插入/删除操作，数据规模不确定

**实现示例**：

```java
public class ListNode {
    int val;
    ListNode next;
    
    public ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}

// 创建链表
ListNode head = new ListNode(1);
head.next = new ListNode(2);
head.next.next = new ListNode(3);
```

#### 栈（Stack）

- **特点**：后进先出（LIFO），只能在栈顶操作
- **时间复杂度**：O(1)
- **适用场景**：括号匹配、函数调用、表达式求值

**实现示例**：

```java
// 使用 LinkedList 实现栈
import java.util.LinkedList;

public class Stack {
    private LinkedList<Integer> list = new LinkedList<>();
    
    public void push(int val) {
        list.addFirst(val);
    }
    
    public int pop() {
        if (list.isEmpty()) {
            throw new RuntimeException("Stack is empty");
        }
        return list.removeFirst();
    }
    
    public int peek() {
        if (list.isEmpty()) {
            throw new RuntimeException("Stack is empty");
        }
        return list.getFirst();
    }
    
    public boolean isEmpty() {
        return list.isEmpty();
    }
}
```

#### 队列（Queue）

- **特点**：先进先出（FIFO），两端操作
- **时间复杂度**：O(1)
- **适用场景**：任务调度、消息队列、广度优先搜索

**实现示例**：

```java
// 使用 LinkedList 实现队列
import java.util.LinkedList;

public class Queue {
    private LinkedList<Integer> list = new LinkedList<>();
    
    public void enqueue(int val) {
        list.addLast(val);
    }
    
    public int dequeue() {
        if (list.isEmpty()) {
            throw new RuntimeException("Queue is empty");
        }
        return list.removeFirst();
    }
    
    public int peek() {
        if (list.isEmpty()) {
            throw new RuntimeException("Queue is empty");
        }
        return list.getFirst();
    }
    
    public boolean isEmpty() {
        return list.isEmpty();
    }
}
```

### 非线性结构

#### 树（Tree）

- **特点**：层次结构，每个节点有零个或多个子节点
- **常见类型**：二叉树、平衡树、B树、B+树
- **适用场景**：搜索、排序、文件系统

**二叉树实现示例**：

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    
    public TreeNode(int val) {
        this.val = val;
        this.left = null;
        this.right = null;
    }
}

// 创建二叉树
TreeNode root = new TreeNode(1);
root.left = new TreeNode(2);
root.right = new TreeNode(3);
root.left.left = new TreeNode(4);
root.left.right = new TreeNode(5);
```

#### 图（Graph）

- **特点**：由顶点和边组成的网络结构
- **常见类型**：有向图、无向图、加权图
- **适用场景**：社交网络、路由算法、推荐系统

**邻接表实现示例**：

```java
import java.util.ArrayList;
import java.util.List;

public class Graph {
    private int V; // 顶点数
    private List<List<Integer>> adj; // 邻接表
    
    public Graph(int V) {
        this.V = V;
        adj = new ArrayList<>(V);
        for (int i = 0; i < V; i++) {
            adj.add(new ArrayList<>());
        }
    }
    
    public void addEdge(int v, int w) {
        adj.get(v).add(w);
        adj.get(w).add(v); // 无向图
    }
    
    public List<Integer> getAdj(int v) {
        return adj.get(v);
    }
}
```

#### 哈希表（Hash Table）

- **特点**：通过哈希函数将键映射到值，快速查找
- **时间复杂度**：平均 O(1)
- **适用场景**：缓存、关联数组、计数器

**实现示例**：

```java
import java.util.LinkedList;

public class HashTable {
    private int capacity;
    private LinkedList<Entry>[] table;
    
    private static class Entry {
        String key;
        int value;
        
        public Entry(String key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    public HashTable(int capacity) {
        this.capacity = capacity;
        table = new LinkedList[capacity];
        for (int i = 0; i < capacity; i++) {
            table[i] = new LinkedList<>();
        }
    }
    
    private int hash(String key) {
        return Math.abs(key.hashCode()) % capacity;
    }
    
    public void put(String key, int value) {
        int index = hash(key);
        LinkedList<Entry> list = table[index];
        
        for (Entry entry : list) {
            if (entry.key.equals(key)) {
                entry.value = value;
                return;
            }
        }
        
        list.add(new Entry(key, value));
    }
    
    public int get(String key) {
        int index = hash(key);
        LinkedList<Entry> list = table[index];
        
        for (Entry entry : list) {
            if (entry.key.equals(key)) {
                return entry.value;
            }
        }
        
        throw new RuntimeException("Key not found");
    }
    
    public void remove(String key) {
        int index = hash(key);
        LinkedList<Entry> list = table[index];
        
        for (Entry entry : list) {
            if (entry.key.equals(key)) {
                list.remove(entry);
                return;
            }
        }
    }
}
```

## 数据结构选择指南

| 操作类型 | 推荐数据结构 | 时间复杂度 |
|---------|------------|-----------|
| 随机访问 | 数组 | O(1) |
| 插入/删除（中间） | 链表 | O(1) |
| 后进先出 | 栈 | O(1) |
| 先进先出 | 队列 | O(1) |
| 有序数据 | 二叉搜索树 | O(log n) |
| 键值对查找 | 哈希表 | O(1) |
| 网络关系 | 图 | 取决于实现 |

## 数据结构应用场景

### 数组
- 存储固定大小的相同类型元素
- 实现矩阵、向量等数学结构
- 作为其他数据结构的基础

### 链表
- 实现栈、队列等抽象数据类型
- 处理动态变化的数据集合
- 内存管理中的自由链表

### 栈
- 函数调用栈
- 表达式求值
- 括号匹配
- 深度优先搜索

### 队列
- 任务调度
- 消息队列
- 广度优先搜索
- 缓冲区管理

### 树
- 二叉搜索树：高效查找、插入、删除
- AVL树：平衡二叉搜索树
- B树/B+树：数据库索引
- 堆：优先队列

### 图
- 社交网络分析
- 路由算法
- 推荐系统
- 网络流问题

### 哈希表
- 缓存实现
- 关联数组
- 计数器
- 去重操作

## 总结

数据结构是计算机科学的基础，选择合适的数据结构对于提高算法效率至关重要。不同的数据结构适用于不同的场景，了解各种数据结构的特点和适用场景，可以帮助我们设计出更高效、更可维护的程序。

在实际应用中，我们常常需要根据具体问题的特点，选择或组合使用不同的数据结构，以达到最佳的性能和空间利用率。