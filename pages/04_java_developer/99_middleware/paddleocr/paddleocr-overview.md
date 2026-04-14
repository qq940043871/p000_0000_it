# PaddleOCR 集成文档

## 1. 技术栈

- **PaddleOCR**：百度开源的OCR工具库，支持多种语言和场景的文字识别
- **Java**：后端开发语言
- **Spring Boot**：Web框架，用于提供HTTP接口
- **Maven**：依赖管理工具
- **OpenCV**：图像处理库，用于预处理
- **Docker**：容器化部署

## 2. 功能介绍

### 2.1 证件识别
- **身份证识别**：识别身份证正面的姓名、性别、民族、出生日期、住址、身份证号等信息
- **护照识别**：识别护照的个人信息页内容
- **驾驶证识别**：识别驾驶证的相关信息

### 2.2 银行卡识别
- **银行卡号识别**：自动识别银行卡号
- **银行卡类型识别**：识别银行卡的类型（借记卡/信用卡）
- **银行名称识别**：识别发卡银行名称

### 2.3 HTTP接口
- 提供RESTful API接口，支持图片上传和识别结果返回
- 支持Base64编码的图片数据
- 支持图片URL传入

## 3. 实现方案

### 3.1 项目结构

```
├── src/main/java
│   ├── com/example/paddleocr
│   │   ├── controller
│   │   │   └── OCRController.java        # HTTP接口控制器
│   │   ├── service
│   │   │   ├── OCRService.java           # OCR服务接口
│   │   │   ├── impl
│   │   │   │   ├── IDCardRecognizer.java  # 身份证识别实现
│   │   │   │   ├── BankCardRecognizer.java # 银行卡识别实现
│   │   │   │   └── OCRServiceImpl.java    # OCR服务实现
│   │   ├── model
│   │   │   ├── IDCardInfo.java            # 身份证信息模型
│   │   │   ├── BankCardInfo.java          # 银行卡信息模型
│   │   │   └── OCRResult.java             # OCR结果模型
│   │   ├── utils
│   │   │   └── ImageUtils.java            # 图像处理工具
│   │   └── config
│   │       └── PaddleOCRConfig.java       # PaddleOCR配置
│   └── resources
│       ├── application.yml                # 应用配置
│       └── models/                        # PaddleOCR模型文件
├── pom.xml                                # Maven依赖配置
└── Dockerfile                             # Docker构建文件
```

### 3.2 核心依赖

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

### 3.3 配置文件

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

## 4. 部署方式

### 4.1 本地部署

1. 克隆项目代码
2. 下载PaddleOCR模型文件到 `src/main/resources/models/` 目录
3. 执行 `mvn clean package` 构建项目
4. 运行 `java -jar target/paddleocr-service.jar` 启动服务

### 4.2 Docker部署

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

## 5. 接口说明

### 5.1 身份证识别接口

- **URL**: `/api/ocr/idcard`
- **Method**: POST
- **Request Body**:
  ```json
  {
    "image": "base64编码的图片数据",
    "imageUrl": "图片URL（可选）"
  }
  ```
- **Response**:
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

### 5.2 银行卡识别接口

- **URL**: `/api/ocr/bankcard`
- **Method**: POST
- **Request Body**:
  ```json
  {
    "image": "base64编码的图片数据",
    "imageUrl": "图片URL（可选）"
  }
  ```
- **Response**:
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

## 6. 性能优化

1. **模型优化**：使用轻量级模型，如PP-OCRv3的移动端模型
2. **并发处理**：使用线程池处理OCR请求
3. **缓存机制**：对相同图片的识别结果进行缓存
4. **批处理**：支持批量图片识别
5. **硬件加速**：在支持GPU的环境中启用GPU加速

## 7. 安全性

1. **输入验证**：对上传的图片进行大小和格式验证
2. **防止恶意请求**：实现请求频率限制
3. **数据脱敏**：对识别结果中的敏感信息进行脱敏处理
4. **HTTPS**：使用HTTPS协议传输数据
5. **权限控制**：对接口访问进行权限控制

## 8. 扩展与维护

1. **模型更新**：定期更新PaddleOCR模型以提高识别准确率
2. **支持更多证件类型**：可扩展支持其他类型的证件识别
3. **多语言支持**：可扩展支持多语言文字识别
4. **监控与告警**：实现服务监控和异常告警机制
5. **日志记录**：记录识别请求和结果，便于问题排查
