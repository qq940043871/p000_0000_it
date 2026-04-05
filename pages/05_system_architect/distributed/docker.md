Docker容器 - 架构师学习笔记
    
    


    
        
            
                Docker容器
            
            
            
                
                    Docker是一个开源的应用容器引擎，基于Go语言并遵从Apache2.0协议开源。Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。
                
                
                Docker核心概念
                
                
                    
                        
                            
                            镜像（Image）
                        
                        
                            Docker镜像是一个轻量级、独立的可执行软件包，包含运行应用程序所需的所有内容：代码、运行时、系统工具、系统库和设置。
                        
                        
                            只读模板
                            可以基于其他镜像创建
                            通过Dockerfile构建
                        
                    
                    
                    
                        
                            
                            容器（Container）
                        
                        
                            容器是镜像的运行实例，可以被启动、开始、停止、删除。每个容器都是相互隔离的、安全的平台。
                        
                        
                            镜像的运行时实例
                            具有状态和生命周期
                            可以被创建、启动、停止、删除
                        
                    
                    
                    
                        
                            
                            仓库（Registry）
                        
                        
                            集中存放镜像文件的场所，分为公开仓库（如Docker Hub）和私有仓库。
                        
                        
                            存储和分发Docker镜像
                            分为公有和私有仓库
                            Docker Hub是最常用的公共仓库
                        
                    
                    
                    
                        
                            
                            Dockerfile
                        
                        
                            用于定义构建Docker镜像的文本文件，包含一系列指令来自动化构建镜像。
                        
                        
                            自动化构建镜像
                            包含一系列构建指令
                            支持版本控制
                        
                    
                
                
                常用Docker命令
                
                
                    
                        镜像操作
                        
# 拉取镜像
docker pull nginx

# 查看本地镜像
docker images

# 构建镜像
docker build -t myapp:1.0 .

# 删除镜像
docker rmi nginx

# 推送镜像到仓库
docker push username/myapp:1.0
                    
                    
                    
                        容器操作
                        
# 运行容器
docker run -d -p 8080:80 nginx

# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 停止容器
docker stop container_id

# 删除容器
docker rm container_id

# 进入容器
docker exec -it container_id /bin/bash
                    
                
                
                Dockerfile详解
                
                
                    常用指令
                    
                        
                            
                                
                                    指令
                                    说明
                                    示例
                                
                            
                            
                                
                                    FROM
                                    指定基础镜像
                                    FROM openjdk:8-jdk-alpine
                                
                                
                                    WORKDIR
                                    设置工作目录
                                    WORKDIR /app
                                
                                
                                    COPY
                                    复制文件到镜像
                                    COPY . /app
                                
                                
                                    ADD
                                    复制文件，支持URL和解压
                                    ADD app.jar /app/app.jar
                                
                                
                                    RUN
                                    执行命令
                                    RUN mvn clean package
                                
                                
                                    EXPOSE
                                    声明端口
                                    EXPOSE 8080
                                
                                
                                    CMD
                                    容器启动时执行的命令
                                    CMD ["java", "-jar", "app.jar"]
                                
                                
                                    ENTRYPOINT
                                    容器启动时执行的命令
                                    ENTRYPOINT ["java", "-jar", "app.jar"]
                                
                            
                        
                    
                    
                    示例Dockerfile
                    
# 使用官方Java运行时作为基础镜像
FROM openjdk:8-jdk-alpine

# 设置工作目录
WORKDIR /app

# 将jar文件复制到容器中
COPY target/myapp.jar app.jar

# 暴露端口
EXPOSE 8080

# 运行应用程序
ENTRYPOINT ["java", "-jar", "app.jar"]
                
                
                Docker网络
                
                
                    
                        网络模式
                        
                            bridge：默认网络模式，为容器分配IP
                            host：容器使用宿主机网络
                            none：容器无网络
                            container：与其他容器共享网络
                        
                    
                    
                    
                        网络操作
                        
# 创建自定义网络
docker network create mynetwork

# 查看网络
docker network ls

# 连接容器到网络
docker network connect mynetwork container_id

# 断开容器网络
docker network disconnect mynetwork container_id
                    
                
                
                数据持久化
                
                
                    
                        
                            
                            数据卷（Volume）
                        
                        
                            Docker管理宿主机文件系统的一部分，由Docker管理，持久化存储数据。
                        
                        
# 创建数据卷
docker volume create myvolume

# 查看数据卷
docker volume ls

# 运行容器并挂载数据卷
docker run -v myvolume:/data nginx
                    
                    
                    
                        
                            
                            绑定挂载（Bind Mount）
                        
                        
                            将宿主机上的文件或目录挂载到容器中，由用户管理。
                        
                        
# 绑定挂载
docker run -v /host/path:/container/path nginx

# 只读挂载
docker run -v /host/path:/container/path:ro nginx
                    
                
                
                Docker Compose
                
                
                    多容器应用编排
                    
                        Docker Compose是用于定义和运行多容器Docker应用程序的工具。
                    
                    
                    
# docker-compose.yml示例
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "80:80"
    depends_on:
      - db
    volumes:
      - ./html:/usr/share/nginx/html
  
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
                    
                    
# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs
                
                
                Docker最佳实践
                
                    
                        使用.dockerignore文件排除不必要的文件
                        合理使用多阶段构建减小镜像体积
                        为镜像打标签，便于版本管理
                        使用非root用户运行容器
                        定期清理无用的镜像和容器
                        使用健康检查确保容器正常运行
                        配置资源限制防止容器耗尽系统资源
