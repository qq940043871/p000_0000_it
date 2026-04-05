Docker实战 - 架构师学习笔记
    
    


    
        
            
                Docker实战
            
            
            
                
                    Docker作为容器化技术的代表，已经成为现代应用部署的标准。掌握Docker的实战技能对于架构师来说至关重要。
                
                
                
                    
                        Docker通过容器化技术解决了应用部署环境不一致的问题，实现了"一次构建，到处运行"的理念。
                    
                
                
                Docker核心概念
                
                
                    
                        
                            镜像(Image)
                        
                        
                            镜像是Docker容器的基础，包含了运行应用所需的所有内容，包括代码、运行时、库、环境变量和配置文件。
                        
                    
                    
                    
                        
                            容器(Container)
                        
                        
                            容器是镜像的运行实例，可以被启动、停止、删除。每个容器都是相互隔离的安全应用平台。
                        
                    
                    
                    
                        
                            网络(Network)
                        
                        
                            Docker网络允许容器之间以及容器与宿主机之间进行通信，支持多种网络驱动。
                        
                    
                    
                    
                        
                            数据卷(Volume)
                        
                        
                            数据卷用于持久化存储容器中的数据，即使容器被删除，数据也不会丢失。
                        
                    
                
                
                常用Docker命令
                
                
                    
                        镜像操作
                        
# 拉取镜像
docker pull nginx:latest

# 列出本地镜像
docker images

# 构建镜像
docker build -t myapp:latest .

# 删除镜像
docker rmi nginx:latest

# 查看镜像详情
docker inspect nginx:latest
                    
                    
                    
                        容器操作
                        
# 运行容器
docker run -d --name mynginx -p 8080:80 nginx:latest

# 列出运行中的容器
docker ps

# 列出所有容器
docker ps -a

# 停止容器
docker stop mynginx

# 启动容器
docker start mynginx

# 删除容器
docker rm mynginx

# 进入容器
docker exec -it mynginx /bin/bash
                    
                    
                    
                        网络和数据卷操作
                        
# 创建网络
docker network create mynetwork

# 列出网络
docker network ls

# 创建数据卷
docker volume create myvolume

# 列出数据卷
docker volume ls

# 挂载数据卷运行容器
docker run -d --name myapp -v myvolume:/data myapp:latest
                    
                
                
                Dockerfile编写
                
                
                    
                        基础Dockerfile示例
                        
FROM openjdk:11-jre-slim

# 设置工作目录
WORKDIR /app

# 复制jar文件到容器中
COPY target/myapp.jar app.jar

# 暴露端口
EXPOSE 8080

# 设置环境变量
ENV JAVA_OPTS=""

# 启动命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
                        这是一个简单的Spring Boot应用Dockerfile示例。
                    
                    
                    
                        多阶段构建
                        
# 构建阶段
FROM maven:3.8.4-openjdk-11 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# 运行阶段
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
                        多阶段构建可以减小最终镜像的大小，提高安全性。
                    
                
                
                Docker Compose
                
                
                    
                        docker-compose.yml示例
                        
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - db
      - redis
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"
      
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  db_data:
                        使用Docker Compose可以定义和运行多容器Docker应用程序。
                    
                    
                    
                        常用Compose命令
                        
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看服务日志
docker-compose logs web

# 停止所有服务
docker-compose down

# 重新构建服务
docker-compose build

# 执行命令
docker-compose exec web /bin/bash
                    
                
                
                
                    Docker最佳实践
                    
                        使用.dockerignore文件排除不必要的文件
                        合理使用多阶段构建减小镜像大小
                        为镜像设置合适的标签
                        避免在容器中存储数据，使用数据卷
                        定期清理无用的镜像和容器
                        使用健康检查确保容器正常运行
