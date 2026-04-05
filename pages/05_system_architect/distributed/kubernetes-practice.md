Kubernetes实战 - 架构师学习笔记
    
    


    
        
            
                Kubernetes实战
            
            
            
                
                    Kubernetes是容器编排的事实标准，掌握Kubernetes实战技能对于构建和管理大规模容器化应用至关重要。
                
                
                
                    
                        Kubernetes提供了自动化的容器部署、扩展和管理功能，是云原生应用的核心基础设施。
                    
                
                
                Kubernetes核心概念
                
                
                    
                        
                            Pod
                        
                        
                            Kubernetes中最小的部署单元，可以包含一个或多个容器，共享网络和存储资源。
                        
                    
                    
                    
                        
                            Deployment
                        
                        
                            用于管理Pod的声明式更新，支持滚动更新、回滚等操作。
                        
                    
                    
                    
                        
                            Service
                        
                        
                            为Pod提供稳定的网络访问入口，实现服务发现和负载均衡。
                        
                    
                    
                    
                        
                            Volume
                        
                        
                            为Pod提供持久化存储，支持多种存储类型。
                        
                    
                
                
                常用kubectl命令
                
                
                    
                        资源管理
                        
# 创建资源
kubectl apply -f deployment.yaml

# 获取资源信息
kubectl get pods
kubectl get deployments
kubectl get services

# 查看资源详情
kubectl describe pod 

# 删除资源
kubectl delete -f deployment.yaml
kubectl delete pod 
                    
                    
                    
                        Pod操作
                        
# 进入Pod容器
kubectl exec -it  -- /bin/bash

# 查看Pod日志
kubectl logs 
kubectl logs -f 

# 复制文件到Pod
kubectl cp  :

# 重启Pod
kubectl delete pod 
                    
                    
                    
                        配置和上下文
                        
# 查看当前上下文
kubectl config current-context

# 切换上下文
kubectl config use-context 

# 查看集群信息
kubectl cluster-info

# 查看节点信息
kubectl get nodes
                    
                
                
                Deployment配置示例
                
                
                    
                        基础Deployment
                        
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
                        这是一个简单的Nginx Deployment配置示例。
                    
                    
                    
                        带健康检查的Deployment
                        
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: springboot
  template:
    metadata:
      labels:
        app: springboot
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
                        添加健康检查可以确保应用的高可用性。
                    
                
                
                Service配置示例
                
                
                    
                        ClusterIP Service
                        
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
                        ClusterIP是默认的Service类型，只能在集群内部访问。
                    
                    
                    
                        NodePort Service
                        
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
                        NodePort类型的Service可以通过节点IP和指定端口从外部访问。
                    
                
                
                ConfigMap和Secret
                
                
                    
                        ConfigMap
                        
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  application.properties: |
    server.port=8080
    spring.datasource.url=jdbc:mysql://mysql:3306/mydb
    spring.datasource.username=user
    spring.datasource.password=password
                        ConfigMap用于存储非敏感的配置数据。
                    
                    
                    
                        Secret
                        
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: dXNlcg==  # base64编码的"user"
  password: cGFzc3dvcmQ=  # base64编码的"password"
                        Secret用于存储敏感信息，如密码、密钥等。
                    
                
                
                
                    Kubernetes最佳实践
                    
                        为资源设置合适的requests和limits
                        使用标签和注解组织和管理资源
                        实施健康检查确保应用可用性
                        使用命名空间隔离不同环境
                        定期备份etcd数据
                        监控集群状态和资源使用情况
