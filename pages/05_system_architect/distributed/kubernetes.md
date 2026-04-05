Kubernetes - 架构师学习笔记
    
    


    
        
            
                Kubernetes
            
            
            
                
                    Kubernetes（简称K8s）是Google开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它提供了服务发现、负载均衡、存储编排、自动部署和回滚、自动装箱、自我修复、密钥与配置管理等功能。
                
                
                Kubernetes核心概念
                
                
                    
                        
                            
                            Pod
                        
                        
                            Kubernetes中最小的可部署单元，可以包含一个或多个容器，共享网络和存储资源。
                        
                        
                            最小部署单元
                            共享网络和存储
                            生命周期短暂
                        
                    
                    
                    
                        
                            
                            Service
                        
                        
                            为一组Pod提供稳定的网络访问入口，实现服务发现和负载均衡。
                        
                        
                            稳定访问入口
                            负载均衡
                            服务发现
                        
                    
                    
                    
                        
                            
                            Deployment
                        
                        
                            用于管理Pod和ReplicaSet，提供声明式的更新和部署能力。
                        
                        
                            声明式部署
                            滚动更新
                            回滚支持
                        
                    
                    
                    
                        
                            
                            Volume
                        
                        
                            为Pod提供持久化存储，支持多种存储类型。
                        
                        
                            持久化存储
                            多种存储类型
                            数据共享
                        
                    
                
                
                Kubernetes架构
                
                
                    控制平面组件
                    
                        
                            
                                
                                    组件
                                    功能
                                
                            
                            
                                
                                    kube-apiserver
                                    集群控制的前端，提供REST API
                                
                                
                                    etcd
                                    分布式键值存储，保存集群状态
                                
                                
                                    kube-scheduler
                                    负责Pod调度到合适的节点
                                
                                
                                    kube-controller-manager
                                    运行控制器进程
                                
                                
                                    cloud-controller-manager
                                    与云提供商交互的控制器
                                
                            
                        
                    
                
                
                
                    节点组件
                    
                        
                            
                                
                                    组件
                                    功能
                                
                            
                            
                                
                                    kubelet
                                    负责节点上的Pod管理
                                
                                
                                    kube-proxy
                                    实现服务网络代理和负载均衡
                                
                                
                                    容器运行时
                                    负责运行容器（如Docker、containerd）
                                
                            
                        
                    
                
                
                常用kubectl命令
                
                
                    
                        资源管理
                        
# 创建资源
kubectl apply -f deployment.yaml

# 删除资源
kubectl delete deployment myapp

# 查看资源
kubectl get pods
kubectl get deployments
kubectl get services

# 查看资源详情
kubectl describe pod pod-name
                    
                    
                    
                        Pod操作
                        
# 进入Pod
kubectl exec -it pod-name -- /bin/bash

# 查看日志
kubectl logs pod-name

# 复制文件到Pod
kubectl cp local-file pod-name:/path/to/file

# 端口转发
kubectl port-forward pod-name 8080:80
                    
                
                
                Deployment示例
                
                
                    
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
                
                
                Service示例
                
                
                    
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
  type: LoadBalancer
                
                
                存储管理
                
                
                    
                        PersistentVolume
                        
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
                    
                    
                    
                        PersistentVolumeClaim
                        
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
                    
                
                
                配置管理
                
                
                    
                        ConfigMap
                        
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: postgresql://localhost:5432/mydb
  log_level: INFO
                    
                    
                    
                        Secret
                        
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: bXl1c2Vy
  password: bXlwYXNzd29yZA==
                    
                
                
                Kubernetes网络
                
                
                    网络模型
                    
                        Pod间通信：所有Pod都在一个可以直接连通的、扁平的网络空间中
                        Pod与Service通信：通过Service的ClusterIP实现
                        外部与Service通信：通过NodePort、LoadBalancer或Ingress实现
                    
                    
                    Ingress示例
                    
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
                
                
                Kubernetes最佳实践
                
                    
                        合理设置资源请求和限制，避免资源浪费
                        使用健康检查（liveness和readiness probes）确保应用健康
                        实施标签和注解策略，便于资源管理和查询
                        使用命名空间隔离不同环境和团队的资源
                        定期备份etcd数据，确保集群状态可恢复
                        实施安全策略，如RBAC、网络策略等
                        监控和日志收集，使用Prometheus、Grafana等工具
