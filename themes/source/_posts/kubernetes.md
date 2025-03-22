---
title: Kubernetes
tags: kubernetes
categories: kubernetes
---

# [Kubernetes](https://www.kubernetes.org.cn/k8s)

## 安装

## 示例部署

### 部署业务应用（deployment 资源）

1. 创建文件 deploy.yaml 如下
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
      namespace: handson-7609b5511e618e0aebc8dcf2c96e23e6
      labels:
        app: myapp
    spec:
      replicas: 1 # 表示当前应用只部署一份
      selector:
        matchLabels:
        name: myapp
      template:
        metadata:
          labels:
          name: myapp
          namespace: handson-7609b5511e618e0aebc8dcf2c96e23e6
        spec:
          containers:
            - name: myapp
              image: registry.cn-shanghai.aliyuncs.com/workbench_1459088147016887/handson_ack_test:3 # web 应用的镜像地址
              ports:
                - containerPort: 8080
    ```

2. 部署
    ```bash
    kubectl apply -f deploy.yaml
    ```

3. 查询已部署的 pod
    ```bash
    kubectl get pod
    ```

4. 重启
    ```bash
    kubectl delete pod myapp-deployment
    # 执行 delete pod 后将会立即重启了一个新的同类型的 pod，因为 deployment 控制 replicasSet，当副本数跟实际的不相符的时候，rs 会自动再创建新的，直到跟模板上的一致
    ```

5. 卸载
    ```bash
    kubectl delete deployment myapp-deployment
    ```

### 部署服务（service 资源）

1. 创建文件 service.yaml 如下
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: myapp-service
      namespace: handson-7609b5511e618e0aebc8dcf2c96e23e6
    spec:
      ports: # 这里定义了服务自身暴露的端口和需要访问的应用的端口
        - port: 8080
          targetPort: 8080
          protocol: TCP
      type: NodePort
      selector: # 这是一个选择器，通过 name=myapp 这个条件来选择需要代理的服务
        name: myapp
    ```

2. 部署
    ```bash
    kubectl apply -f service.yaml
    ```

3. 查询已部署的 service
    ```bash
    kubectl get service
    ```

4. 卸载
    ```bash
    kubectl delete service myapp-service
    ```

### 配置 ingress 开放外部访问

1. 创建文件 ingress.yaml 如下
    ```yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: example-ingress
      namespace: handson-7609b5511e618e0aebc8dcf2c96e23e6
    spec:
      rules:
        - http:
            paths:
              - path: /welcome
                backend:
                  serviceName: myapp-service
                  servicePort: 8080
    ```

2. 部署
    ```bash
    kubectl apply -f ingress.yaml
    ```

3. 查询已部署的 ingress
    ```bash
    kubectl get ingress
    ```

4. 卸载
    ```bash
    kubectl delete ingress example-ingress
    ```

## kubectl 命令

```bash
# 查看所有命名空间
kubectl get namespace

# 查看默认命名空间 default 下所有 pods
kubectl get pods
# 查看命名空间 n1 下所有 pods
kubectl get pods -n n1

# 查看当前可用的 api 版本
kubectl api-versions

# 查看命名空间 n1 下 example-ingress 的描述 yaml 文件
kubectl -n n1 get ingress -o yaml example-ingress
```

## yaml 配置

```yaml
# 必填，api 版本，可通过命令 kubectl api-versions 查询所有可选值
apiVersion: v1

# 必填，资源类别，包括：
#   资源对象 Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling
#   配置对象 Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、ThirdPartyResource、ServiceAccount
#   存储对象 Volume、Persistent Volume
#   策略对象 SecurityContext、ResourceQuota、LimitRange
kind: Pod

metadata: # 必填，元数据
  name: string # 必填，名称
  namespace: string # 必填，所属命名空间
  labels: # 自定义标签
    - name: string # 自定义标签名字
  annotations: # 自定义注释列表
    - name: string
spec: # 必选，容器的详细定义
  containers: # 必选，Pod 中容器列表
  - name: string # 必选，容器名称
    image: string # 必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent] # 获取镜像的策略，Alawys 表示下载镜像，IfnotPresent 表示优先使用本地镜像，否则下载镜像，Nerver 表示仅使用本地镜像
    command: [string] # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string] # 容器的启动命令参数列表
    workingDir: string # 容器的工作目录
    volumeMounts: # 挂载到容器内部的存储卷配置
    - name: string # 引用 pod 定义的共享存储卷的名称，需用 volumes[] 部分定义的的卷名
      mountPath: string # 存储卷在容器内 mount 的绝对路径，应少于 512 字符
      readOnly: boolean # 是否为只读模式
    ports: # 需要暴露的端口库号列表
    - name: string # 端口号名称
      containerPort: int # 容器需要监听的端口号
      hostPort: int # 容器所在主机需要监听的端口号，默认与 Container 相同
      protocol: string # 端口协议，支持 TCP 和 UDP，默认 TCP
    env: # 容器运行前需设置的环境变量列表
    - name: string # 环境变量名称
      value: string # 环境变量的值
    resources: # 资源限制和请求的设置
      limits: # 资源限制的设置
        cpu: string # Cpu 的限制，单位为 core 数，将用于 docker run --cpu-shares 参数
        memory: string # 内存限制，单位可以为 Mib/Gib，将用于 docker run --memory 参数
      requests: # 资源请求的设置
        cpu: string # Cpu 请求，容器启动的初始可用数量
        memory: string # 内存清楚，容器启动的初始可用数量
    livenessProbe: # 对 Pod 内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有 exec、httpGet 和 tcpSocket，对一个容器只需设置其中一种方法即可
      exec: # 对Pod容器内检查方式设置为 exec 方式
        command: [string] # exec 方式需要制定的命令或脚本
      httpGet: # 对 Pod 内个容器健康检查方法设置为 HttpGet，需要制定 Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket: # 对 Pod 内个容器健康检查方式设置为 tcpSocket 方式
         port: number
       initialDelaySeconds: 0 # 容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0 # 对容器健康检查探测等待响应的超时时间，单位秒，默认 1 秒
       periodSeconds: 0 # 对容器监控检查的定期探测时间设置，单位秒，默认 10 秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
    restartPolicy: [Always | Never | OnFailure] # Pod 的重启策略，Always 表示一旦不管以何种方式终止运行，kubelet 都将重启，OnFailure 表示只有 Pod 以非 0 退出码退出才重启，Nerver 表示不再重启该 Pod
    nodeSelector: obeject # 设置 NodeSelector 表示将该 Pod 调度到包含这个 label 的 node 上，以 key: value 的格式指定
    imagePullSecrets: # Pull 镜像时使用的 secret 名称，以 key: secretkey 格式指定
    - name: string
    hostNetwork:false # 是否使用主机网络模式，默认为 false，如果设置为 true，表示使用宿主机网络
    volumes: # 在该 pod 上定义共享存储卷列表
    - name: string # 共享存储卷名称（volumes 类型有很多种）
      emptyDir: {} # 类型为 emtyDir 的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string # 类型为 hostPath 的存储卷，表示挂载 Pod 所在宿主机的目录
        path: string # Pod 所在宿主机的目录，将被用于同期中 mount 的目录
      secret: # 类型为 secret 的存储卷，挂载集群与定义的 secre 对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap: # 类型为 configMap 的存储卷，挂载预定义的 configMap 对象到容器内部
        name: string
        items:
        - key: string
          path: string
```
