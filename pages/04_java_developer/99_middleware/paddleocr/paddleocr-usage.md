# PaddleOCR 使用文档

## 1. 环境搭建

### 1.1 系统要求
- **操作系统**：Windows 10/11、Linux、macOS
- **Java 版本**：JDK 8 及以上
- **内存**：至少 4GB
- **磁盘空间**：至少 1GB（用于存储模型文件）

### 1.2 依赖安装

#### Maven 依赖配置

在 `pom.xml` 文件中添加以下依赖：

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- PaddleOCR Java SDK -->
    <dependency>
        <groupId>com.baidu.paddle</groupId>
        <artifactId>paddleocr</artifactId>
        <version>2.0.0</version>
    </dependency>
    
    <!-- OpenCV -->
    <dependency>
        <groupId>org.openpnp</groupId>
        <artifactId>opencv</artifactId>
        <version>4.5.5-1</version>
    </dependency>
    
    <!-- Jackson JSON -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

### 1.3 模型下载

PaddleOCR 需要下载以下模型文件：

1. **检测模型**：`ch_PP-OCRv3_det_infer`
2. **识别模型**：`ch_PP-OCRv3_rec_infer`
3. **分类模型**：`ch_PP-OCRv3_cls_infer`

下载链接：[PaddleOCR 模型库](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.6/doc/doc_ch/models_list.md)

下载完成后，将模型文件解压到 `src/main/resources/models/` 目录下。

## 2. 项目配置

### 2.1 应用配置

在 `application.yml` 文件中添加以下配置：

```yaml
spring:
  application:
    name: paddleocr-service
  
server:
  port: 8080

paddleocr:
  model-dir: classpath:models/
  det-model: ch_PP-OCRv3_det_infer
  rec-model: ch_PP-OCRv3_rec_infer
  cls-model: ch_PP-OCRv3_cls_infer
  use-gpu: false
  gpu-memory: 4000
```

### 2.2 环境变量配置

如果使用 GPU 加速，需要配置以下环境变量：

- **Windows**：设置 `CUDA_HOME` 环境变量，指向 CUDA 安装目录
- **Linux**：在 `.bashrc` 文件中添加 `export CUDA_HOME=/path/to/cuda`

## 3. 接口调用

### 3.1 身份证识别接口

#### 请求方式
- **URL**：`/api/ocr/idcard`
- **Method**：POST
- **Content-Type**：`application/json`

#### 请求参数

| 参数名 | 类型 | 必选 | 说明 |
|--------|------|------|------|
| image | String | 否 | Base64 编码的图片数据 |
| imageUrl | String | 否 | 图片 URL 地址 |

**注意**：`image` 和 `imageUrl` 必须二选一

#### 请求示例

```json
{
  "image": "iVBORw0KGgoAAAANSUhEUgAA..."  // Base64 编码的图片数据
}
```

或

```json
{
  "imageUrl": "https://example.com/idcard.jpg"
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "name": "张三",
    "gender": "男",
    "nation": "汉",
    "birth": "19900101",
    "address": "北京市朝阳区",
    "idNumber": "110101199001011234"
  }
}
```

### 3.2 银行卡识别接口

#### 请求方式
- **URL**：`/api/ocr/bankcard`
- **Method**：POST
- **Content-Type**：`application/json`

#### 请求参数

| 参数名 | 类型 | 必选 | 说明 |
|--------|------|------|------|
| image | String | 否 | Base64 编码的图片数据 |
| imageUrl | String | 否 | 图片 URL 地址 |

**注意**：`image` 和 `imageUrl` 必须二选一

#### 请求示例

```json
{
  "image": "iVBORw0KGgoAAAANSUhEUgAA..."  // Base64 编码的图片数据
}
```

或

```json
{
  "imageUrl": "https://example.com/bankcard.jpg"
}
```

#### 响应示例

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "cardNumber": "6222021234567890123",
    "bankName": "中国工商银行",
    "cardType": "借记卡"
  }
}
```

## 4. 客户端调用示例

### 4.1 Java 客户端

```java
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.util.Base64;
import java.nio.file.Files;
import java.nio.file.Paths;

public class OCRClient {
    public static void main(String[] args) throws IOException, InterruptedException {
        // 读取图片并转换为Base64
        byte[] imageData = Files.readAllBytes(Paths.get("test-idcard.jpg"));
        String base64Image = Base64.getEncoder().encodeToString(imageData);
        
        // 构建请求体
        String requestBody = "{\"image\": \"" + base64Image + "\"}";
        
        // 发送请求
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:8080/api/ocr/idcard"))
                .header("Content-Type", "application/json")
                .POST(HttpRequest.BodyPublishers.ofString(requestBody))
                .build();
        
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
    }
}
```

### 4.2 Python 客户端

```python
import requests
import base64

# 读取图片并转换为Base64
with open('test-idcard.jpg', 'rb') as f:
    image_data = f.read()
base64_image = base64.b64encode(image_data).decode('utf-8')

# 构建请求体
payload = {
    'image': base64_image
}

# 发送请求
response = requests.post('http://localhost:8080/api/ocr/idcard', json=payload)
print(response.json())
```

### 4.3 curl 命令

```bash
# 读取图片并转换为Base64
BASE64_IMAGE=$(base64 -i test-idcard.jpg)

# 发送请求
curl -X POST http://localhost:8080/api/ocr/idcard \
  -H "Content-Type: application/json" \
  -d "{\"image\": \"$BASE64_IMAGE\"}"
```

## 5. 常见问题

### 5.1 模型文件缺失

**问题**：启动服务时提示模型文件缺失

**解决方案**：
1. 确保模型文件已下载并解压到 `src/main/resources/models/` 目录
2. 检查 `application.yml` 中的模型配置路径是否正确

### 5.2 识别准确率低

**问题**：识别结果准确率不高

**解决方案**：
1. 确保图片清晰，光线充足
2. 调整图片角度，确保证件或银行卡处于水平位置
3. 使用更高精度的模型
4. 对图片进行预处理，如调整亮度、对比度等

### 5.3 服务启动失败

**问题**：服务启动时出现异常

**解决方案**：
1. 检查 Java 版本是否为 JDK 8 及以上
2. 检查依赖是否正确配置
3. 检查端口是否被占用
4. 查看日志获取详细错误信息

### 5.4 GPU 加速不生效

**问题**：配置了 GPU 加速但未生效

**解决方案**：
1. 确保安装了 CUDA 和 cuDNN
2. 检查环境变量配置是否正确
3. 检查 PaddleOCR 是否支持当前 CUDA 版本
4. 确保 `use-gpu` 配置为 `true`

## 6. 性能优化

### 6.1 模型优化
- 使用轻量级模型，如 PP-OCRv3 的移动端模型
- 对模型进行量化，减少模型大小和推理时间

### 6.2 服务优化
- 使用线程池处理并发请求
- 实现请求缓存，避免重复识别相同图片
- 调整 JVM 参数，优化内存使用

### 6.3 客户端优化
- 压缩图片后再上传，减少网络传输时间
- 使用异步请求，提高并发处理能力
- 实现本地缓存，避免重复请求

## 7. 安全建议

1. **输入验证**：对上传的图片进行大小和格式验证，防止恶意文件上传
2. **请求限流**：实现请求频率限制，防止恶意请求攻击
3. **数据脱敏**：对识别结果中的敏感信息进行脱敏处理
4. **HTTPS**：使用 HTTPS 协议传输数据，确保数据安全
5. **权限控制**：对接口访问进行权限控制，只允许授权用户访问

## 8. 部署建议

### 8.1 本地部署

1. 克隆项目代码
2. 下载模型文件到 `src/main/resources/models/` 目录
3. 执行 `mvn clean package` 构建项目
4. 运行 `java -jar target/paddleocr-service.jar` 启动服务

### 8.2 Docker 部署

```dockerfile
FROM openjdk:8-jdk-alpine

WORKDIR /app

COPY target/paddleocr-service.jar app.jar
COPY src/main/resources/models/ models/

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

构建和运行：

```bash
docker build -t paddleocr-service .
docker run -p 8080:8080 paddleocr-service
```

### 8.3 容器编排

使用 Kubernetes 进行容器编排，实现服务的自动扩缩容和负载均衡。

## 9. 监控与维护

### 9.1 日志监控
- 配置日志收集，使用 ELK 或 Loki 等工具进行日志分析
- 设置关键操作的日志记录，便于问题排查

### 9.2 性能监控
- 监控服务响应时间和并发处理能力
- 监控系统资源使用情况，如 CPU、内存、磁盘等

### 9.3 异常告警
- 配置异常告警机制，当服务出现异常时及时通知运维人员
- 设置服务健康检查，确保服务正常运行

## 10. 版本更新

### 10.1 模型更新
- 定期更新 PaddleOCR 模型，以提高识别准确率
- 关注 PaddleOCR 官方发布的新模型和技术

### 10.2 代码更新
- 定期更新依赖库，修复安全漏洞
- 优化代码结构，提高代码可维护性
- 添加新功能，如支持更多证件类型的识别

## 11. 总结

PaddleOCR 是一个功能强大的 OCR 工具库，通过本文档的指导，您可以快速实现证件识别和银行卡识别功能，并提供 HTTP 接口供其他系统调用。在实际应用中，您可以根据具体需求进行适当的调整和优化，以达到最佳的识别效果和性能。