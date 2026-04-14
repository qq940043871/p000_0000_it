# XMReport 报表设计器使用文档

## 1. 环境搭建

### 1.1 系统要求
- **操作系统**：Windows 10/11、Linux、macOS
- **Java 版本**：JDK 8 及以上
- **Node.js 版本**：16.0 及以上
- **MySQL**：8.0 及以上
- **MinIO**：2023.0 及以上
- **Docker**：（可选）用于容器化部署

### 1.2 依赖安装

#### 后端依赖

1. 克隆项目代码
2. 进入 `xmreport-backend` 目录
3. 执行 `mvn clean install` 安装依赖

#### 前端依赖

1. 进入 `xmreport-frontend` 目录
2. 执行 `npm install` 安装依赖

### 1.3 服务启动

#### MySQL 启动

```bash
# 启动 MySQL 服务
sudo systemctl start mysql

# 创建数据库
mysql -u root -p
CREATE DATABASE xmreport CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

#### MinIO 启动

```bash
# 启动 MinIO 服务
minio server /data --console-address :9001

# 创建存储桶
mc alias set myminio http://localhost:9000 minioadmin minioadmin
mc mb myminio/xmreport
```

#### 后端服务启动

```bash
# 进入 xmreport-backend 目录
cd xmreport-backend

# 构建项目
mvn clean package

# 启动服务
java -jar target/xmreport-service.jar
```

#### 前端服务启动

```bash
# 进入 xmreport-frontend 目录
cd xmreport-frontend

# 启动开发服务
npm run dev

# 构建生产版本
npm run build
```

## 2. 模板配置

### 2.1 上传模板

1. 登录 XMReport 系统
2. 点击左侧导航栏的「模板管理」
3. 点击「上传模板」按钮
4. 选择 DOCX 模板文件
5. 填写模板名称和描述
6. 点击「保存」按钮

### 2.2 编辑模板

1. 在模板列表中找到要编辑的模板
2. 点击「编辑」按钮
3. 在模板编辑器中修改模板内容
4. 使用 `${variable}` 形式添加变量占位符
5. 使用 `{{#if condition}}...{{/if}}` 形式添加条件渲染
6. 使用 `{{#each list}}...{{/each}}` 形式添加循环渲染
7. 点击「预览」按钮查看效果
8. 点击「保存」按钮保存模板

### 2.3 模板变量

#### 变量占位符

```
${variable}  # 简单变量
${user.name}  # 嵌套变量
${list[0].name}  # 数组变量
${a + b}  # 表达式
```

#### 条件渲染

```
{{#if condition}}
  条件为真时显示的内容
{{/if}}

{{#if condition}}
  条件为真时显示的内容
{{else}}
  条件为假时显示的内容
{{/if}}
```

#### 循环渲染

```
{{#each list}}
  {{this.name}}  # 访问当前元素的属性
{{/each}}

{{#each list as |item index|}}
  {{index}}: {{item.name}}  # 访问索引和当前元素
{{/each}}
```

## 3. 数据源配置

### 3.1 数据库数据源

1. 点击左侧导航栏的「数据源管理」
2. 点击「创建数据源」按钮
3. 选择「数据库数据源」类型
4. 填写数据源名称、描述
5. 选择数据库类型（MySQL、Oracle、PostgreSQL 等）
6. 填写数据库 URL、用户名、密码
7. 点击「测试连接」按钮验证连接
8. 点击「保存」按钮

### 3.2 API 数据源

1. 点击左侧导航栏的「数据源管理」
2. 点击「创建数据源」按钮
3. 选择「API 数据源」类型
4. 填写数据源名称、描述
5. 填写 API URL、请求方法、请求头、请求体
6. 点击「测试连接」按钮验证连接
7. 点击「保存」按钮

### 3.3 Excel 数据源

1. 点击左侧导航栏的「数据源管理」
2. 点击「创建数据源」按钮
3. 选择「Excel 数据源」类型
4. 填写数据源名称、描述
5. 上传 Excel 文件
6. 选择工作表
7. 点击「保存」按钮

## 4. 报表生成

### 4.1 生成报表

1. 点击左侧导航栏的「报表管理」
2. 点击「生成报表」按钮
3. 选择模板
4. 选择数据源
5. 填写报表参数
6. 选择输出格式（PDF、DOCX、Excel、HTML）
7. 点击「生成」按钮
8. 等待报表生成完成
9. 点击「下载」按钮下载报表

### 4.2 报表预览

1. 在报表列表中找到要预览的报表
2. 点击「预览」按钮
3. 在预览窗口中查看报表内容
4. 点击「关闭」按钮关闭预览窗口

### 4.3 报表历史

1. 点击左侧导航栏的「报表管理」
2. 点击「历史记录」标签页
3. 查看报表生成历史
4. 点击「下载」按钮下载历史报表
5. 点击「删除」按钮删除历史报表

## 5. HTTP 接口调用

### 5.1 模板管理接口

#### 上传模板

- **URL**：`/api/templates`
- **Method**：POST
- **Content-Type**：`multipart/form-data`
- **请求参数**：
  - `file`：模板文件（DOCX）
  - `name`：模板名称
  - `description`：模板描述

#### 获取模板列表

- **URL**：`/api/templates`
- **Method**：GET
- **响应示例**：
  ```json
  {
    "code": 200,
    "message": "success",
    "data": [
      {
        "id": 1,
        "name": "员工报表模板",
        "description": "员工信息报表模板",
        "createTime": "2023-07-01T12:00:00"
      }
    ]
  }
  ```

### 5.2 报表生成接口

#### 生成报表

- **URL**：`/api/reports`
- **Method**：POST
- **Content-Type**：`application/json`
- **请求参数**：
  ```json
  {
    "templateId": 1,
    "dataSourceId": 1,
    "parameters": {
      "department": "技术部"
    },
    "format": "pdf"
  }
  ```
- **响应示例**：
  ```json
  {
    "code": 200,
    "message": "success",
    "data": {
      "id": 1,
      "name": "员工报表_20230701_120000",
      "url": "http://localhost:9000/xmreport/reports/employee_report_20230701_120000.pdf"
    }
  }
  ```

### 5.3 数据源管理接口

#### 创建数据源

- **URL**：`/api/datasources`
- **Method**：POST
- **Content-Type**：`application/json`
- **请求参数**：
  ```json
  {
    "name": "MySQL 数据源",
    "description": "公司 MySQL 数据库",
    "type": "database",
    "config": {
      "driver": "com.mysql.cj.jdbc.Driver",
      "url": "jdbc:mysql://localhost:3306/company",
      "username": "root",
      "password": "password"
    }
  }
  ```

## 6. 客户端调用示例

### 6.1 Java 客户端

```java
import java.io.File;
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class XMReportClient {
    public static void main(String[] args) throws IOException, InterruptedException {
        // 生成报表
        generateReport();
    }
    
    private static void generateReport() throws IOException, InterruptedException {
        String url = "http://localhost:8080/api/reports";
        
        // 构建请求体
        Map<String, Object> requestBody = new HashMap<>();
        requestBody.put("templateId", 1);
        requestBody.put("dataSourceId", 1);
        
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("department", "技术部");
        requestBody.put("parameters", parameters);
        
        requestBody.put("format", "pdf");
        
        // 发送请求
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(url))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(new ObjectMapper().writeValueAsString(requestBody)))
                .build();
        
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
    }
}
```

### 6.2 Python 客户端

```python
import requests
import json

# 生成报表
def generate_report():
    url = "http://localhost:8080/api/reports"
    
    # 构建请求体
    request_body = {
        "templateId": 1,
        "dataSourceId": 1,
        "parameters": {
            "department": "技术部"
        },
        "format": "pdf"
    }
    
    # 发送请求
    response = requests.post(url, json=request_body)
    print(response.json())

if __name__ == "__main__":
    generate_report()
```

### 6.3 curl 命令

```bash
# 生成报表
curl -X POST http://localhost:8080/api/reports \
  -H "Content-Type: application/json" \
  -d '{"templateId": 1, "dataSourceId": 1, "parameters": {"department": "技术部"}, "format": "pdf"}'
```

## 7. 常见问题

### 7.1 模板上传失败

**问题**：上传模板时提示失败

**解决方案**：
1. 检查模板文件是否为 DOCX 格式
2. 检查模板文件大小是否超过限制
3. 检查 MinIO 服务是否正常运行
4. 查看后端日志获取详细错误信息

### 7.2 报表生成失败

**问题**：生成报表时提示失败

**解决方案**：
1. 检查模板语法是否正确
2. 检查数据源连接是否正常
3. 检查数据源返回的数据格式是否与模板匹配
4. 查看后端日志获取详细错误信息

### 7.3 数据源连接失败

**问题**：测试数据源连接时提示失败

**解决方案**：
1. 检查数据源配置是否正确
2. 检查网络连接是否正常
3. 检查数据源服务是否正常运行
4. 查看后端日志获取详细错误信息

### 7.4 前端页面加载缓慢

**问题**：前端页面加载缓慢

**解决方案**：
1. 检查网络连接是否正常
2. 检查后端服务是否正常运行
3. 清除浏览器缓存
4. 优化前端代码，减少资源加载

## 8. 性能优化

### 8.1 模板优化
- 减少模板中的变量和表达式数量
- 避免使用复杂的条件和循环
- 优化模板结构，减少嵌套层次

### 8.2 数据源优化
- 优化数据库查询语句，减少查询时间
- 使用索引提高查询效率
- 缓存查询结果，减少重复查询

### 8.3 服务优化
- 调整 JVM 参数，优化内存使用
- 使用线程池处理并发请求
- 配置连接池，提高数据库连接效率

### 8.4 存储优化
- 定期清理过期的报表文件
- 压缩存储的文件，减少存储空间
- 使用对象存储服务，提高存储效率

## 9. 安全建议

1. **输入验证**：对上传的模板和输入参数进行验证，防止恶意文件上传
2. **SQL 注入防护**：使用参数化查询，防止 SQL 注入攻击
3. **XSS 防护**：对用户输入进行转义，防止 XSS 攻击
4. **权限控制**：基于角色的权限控制，限制用户操作
5. **HTTPS**：使用 HTTPS 协议传输数据，确保数据安全
6. **敏感信息加密**：加密存储数据库密码等敏感信息
7. **日志审计**：记录操作日志，便于安全审计和问题排查

## 10. 部署建议

### 10.1 本地开发环境
- 使用 Docker Compose 快速搭建开发环境
- 配置热部署，提高开发效率
- 使用本地数据库和 MinIO 服务

### 10.2 测试环境
- 模拟生产环境配置
- 进行性能测试和安全测试
- 验证报表生成功能的正确性

### 10.3 生产环境
- 使用集群部署，提高服务可用性
- 配置负载均衡，分散请求压力
- 使用备份策略，确保数据安全
- 配置监控告警，及时发现和处理问题

## 11. 监控与维护

### 11.1 日志监控
- 配置日志收集，使用 ELK 或 Loki 等工具进行日志分析
- 设置关键操作的日志记录，便于问题排查

### 11.2 性能监控
- 监控服务响应时间和并发处理能力
- 监控系统资源使用情况，如 CPU、内存、磁盘等
- 监控数据库和 MinIO 服务的性能

### 11.3 异常告警
- 配置异常告警机制，当服务出现异常时及时通知运维人员
- 设置服务健康检查，确保服务正常运行
- 监控报表生成成功率，及时发现问题

## 12. 版本更新

### 12.1 功能更新
- 定期更新系统功能，添加新特性
- 优化用户界面，提高用户体验
- 支持更多模板格式和输出格式

### 12.2 安全更新
- 定期更新依赖库，修复安全漏洞
- 加强安全防护措施，防止安全攻击
- 进行安全审计，确保系统安全

### 12.3 性能更新
- 优化系统性能，提高报表生成速度
- 优化数据库查询，减少查询时间
- 优化存储方案，提高存储效率

## 13. 总结

XMReport 报表设计器是一个功能强大的开源报表工具，支持 DOCX 模板配置和 PDF 生成功能。通过本文档的指导，您可以快速上手使用 XMReport，创建和管理报表模板，配置数据源，生成各种格式的报表。在实际应用中，您可以根据具体需求进行适当的调整和优化，以达到最佳的使用效果。