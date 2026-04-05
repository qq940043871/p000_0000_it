# 负载均衡 - 架构师学习笔记

## 负载均衡

负载均衡是一种计算机技术，用于在多个计算资源（如服务器、网络连接、CPU、磁盘驱动器等）之间分配工作负载。在分布式系统中，负载均衡器位于客户端和服务器之间，将客户端请求分发到多个后端服务器，以提高系统的可用性、可靠性和性能。

## 负载均衡的作用

### 提高系统性能

- 通过将请求分发到多个服务器，避免单个服务器过载，提高整体处理能力。

### 增强系统可用性

- 当某个服务器出现故障时，负载均衡器可以将请求转发到其他正常服务器。

### 支持水平扩展

- 可以动态添加或移除服务器，实现系统的弹性伸缩。

### 优化资源利用率

- 合理分配请求，避免部分服务器空闲而其他服务器过载。

## 负载均衡分类

### 按实现方式分类

#### 硬件负载均衡

- 使用专门的硬件设备实现负载均衡，性能高但成本昂贵。

  - F5 BIG-IP
  - Citrix NetScaler

#### 软件负载均衡

- 基于软件实现的负载均衡，成本低，灵活性高。

  - Nginx
  - HAProxy
  - LVS

#### DNS负载均衡

- 通过DNS解析实现负载均衡，配置简单但控制力弱。

  - 智能DNS
  - GeoDNS

### 按工作层次分类

| 层次 | 特点 | 代表产品 |
|------|------|----------|
| 二层负载均衡 | 基于MAC地址分发 | Ribonsticker |
| 三层负载均衡 | 基于IP地址分发 | LVS |
| 四层负载均衡 | 基于TCP/UDP端口分发 | LVS, F5 |
| 七层负载均衡 | 基于应用层信息分发 | Nginx, HAProxy |

## 负载均衡算法

### 静态负载均衡算法

- 轮询（Round Robin）：按顺序将请求分发给服务器
- 加权轮询（Weighted Round Robin）：根据服务器权重分配请求
- IP哈希（IP Hash）：根据客户端IP地址分配请求

### 动态负载均衡算法

- 最少连接（Least Connections）：将请求分发给当前连接数最少的服务器
- 最快响应（Fastest Response）：将请求分发给响应最快的服务器
- 观察方法（Observed）：基于连接数和响应时间的综合算法

## 主流负载均衡器

### Nginx

- 高性能HTTP和反向代理服务器，支持七层负载均衡。

```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=2;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### HAProxy

- 专业的TCP/HTTP负载均衡器，支持四层和七层负载均衡。

```haproxy
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server server1 192.168.1.10:8080 check
    server server2 192.168.1.11:8080 check
    server server3 192.168.1.12:8080 check
```

### LVS (Linux Virtual Server)

- 基于Linux内核的四层负载均衡，性能极高。

```bash
# ipvsadm配置示例
ipvsadm -A -t 192.168.1.100:80 -s rr
ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.10:8080 -g
ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.11:8080 -g
```

## 健康检查

### 检查方式

#### 主动健康检查

- 负载均衡器定期向后端服务器发送探测请求。

  - TCP连接检查
  - HTTP请求检查
  - 自定义脚本检查

#### 被动健康检查

- 通过实际请求的响应情况判断服务器状态。

  - 连接超时
  - 响应错误
  - 响应时间过长

### Nginx健康检查配置

```nginx
upstream backend {
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    
    # 启用主动健康检查（需要商业版）
    zone backend 64k;
    keepalive 32;
}

# 商业版健康检查配置
match server_ok {
    status 200-399;
    body !~ "maintenance";
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        health_check match=server_ok;
    }
}
```

## 会话保持

### 基于Cookie的会话保持

- 通过在客户端设置Cookie来识别用户，实现会话保持。

```nginx
# Nginx配置
upstream backend {
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

# 或使用sticky模块
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

### 集中式会话管理

- 将会话数据存储在共享存储中，避免会话绑定。

  - Redis集中存储会话
  - 数据库存储会话
  - 共享文件系统

## 负载均衡最佳实践

- 选择合适的负载均衡算法
- 实施完善的健康检查机制
- 配置合理的超时和重试策略
- 启用会话保持（如需要）
- 监控负载均衡器性能
- 定期更新和维护负载均衡器
- 制定故障切换和恢复方案
