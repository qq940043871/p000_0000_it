# 监控与告警 - 架构师学习笔记

## 监控与告警

监控与告警是保障系统稳定运行的关键环节。通过建立完善的监控体系，我们可以及时发现系统异常，预防故障发生。

好的监控系统应该具备全面性、实时性、可操作性和可扩展性，能够帮助团队快速定位和解决问题。

## 监控体系架构

### 指标监控(Metrics)

- 收集系统、应用和服务的性能指标，如CPU使用率、内存占用、请求延迟等。

### 日志监控(Logs)

- 收集和分析应用日志、系统日志，用于故障排查和安全审计。

### 链路追踪(Traces)

- 追踪请求在分布式系统中的完整调用链路，帮助定位性能瓶颈。

### 告警系统(Alerting)

- 基于监控数据设置告警规则，及时通知相关人员处理异常情况。

## Prometheus监控

### Prometheus配置

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
      
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
      
  - job_name: 'springboot-app'
    static_configs:
      - targets: ['app:8080']
```

Prometheus配置文件定义了监控目标和抓取间隔。

### Spring Boot集成Micrometer

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  metrics:
    tags:
      application: ${spring.application.name}
```

通过Micrometer将Spring Boot应用指标暴露给Prometheus。

### 自定义指标

```java
@Component
public class CustomMetrics {
    
    private final Counter requestCounter;
    private final Timer requestTimer;
    
    public CustomMetrics(MeterRegistry meterRegistry) {
        requestCounter = Counter.builder("http.requests")
            .description("HTTP请求计数")
            .register(meterRegistry);
            
        requestTimer = Timer.builder("http.requests.duration")
            .description("HTTP请求耗时")
            .register(meterRegistry);
    }
    
    public void recordRequest() {
        requestCounter.increment();
    }
    
    public Timer.Sample startTimer() {
        return Timer.Sample.start();
    }
    
    public void stopTimer(Timer.Sample sample) {
        sample.stop(requestTimer);
    }
}
```

通过Micrometer创建自定义业务指标。

## Grafana可视化

### Docker部署Grafana

```yaml
version: '3.8'
services:
  grafana:
    image: grafana/grafana-enterprise
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  grafana-storage:
```

使用Docker Compose部署Grafana。

### 常用监控面板查询

```promql
# JVM内存使用情况
jvm_memory_used_bytes{application="$application", instance="$instance"}

# HTTP请求速率
rate(http_requests_total{application="$application", instance="$instance"}[5m])

# HTTP请求延迟(95th percentile)
histogram_quantile(0.95, rate(http_requests_duration_seconds_bucket[5m]))

# 系统CPU使用率
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

常用的PromQL查询语句，用于创建监控面板。

## 告警规则配置

### Prometheus告警规则

```yaml
# 告警规则
groups:
- name: example
  rules:
  # 实例是否存活
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "实例 {{ $labels.instance }} 已停止运行"
      
  # 高CPU使用率
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "实例 {{ $labels.instance }} CPU使用率过高 ({{ $value }}%)"
      
  # 高内存使用率
  - alert: HighMemoryUsage
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "实例 {{ $labels.instance }} 内存使用率过高 ({{ $value }}%)"
```

Prometheus告警规则定义了触发告警的条件。

### Alertmanager配置

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alert@example.com'
  smtp_auth_username: 'alert@example.com'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'team-X'

receivers:
- name: 'team-X'
  email_configs:
  - to: 'team-X@example.com'
    send_resolved: true
```

Alertmanager负责处理告警通知，支持邮件、Slack等多种通知方式。

## 日志收集与分析

### ELK Stack配置

```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
      
  kibana:
    image: docker.elastic.co/kibana/kibana:7.17.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
      
  logstash:
    image: docker.elastic.co/logstash/logstash:7.17.0
    ports:
      - "5044:5044"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
```

使用Docker Compose部署ELK Stack进行日志收集和分析。

## 监控与告警最佳实践

- 建立分层监控体系，覆盖基础设施、应用和服务层面
- 设置合理的告警阈值，避免告警风暴
- 定期审查和优化告警规则
- 建立完善的监控面板，直观展示系统状态
- 实施日志标准化，便于分析和排查问题
- 定期进行故障演练，验证监控告警有效性