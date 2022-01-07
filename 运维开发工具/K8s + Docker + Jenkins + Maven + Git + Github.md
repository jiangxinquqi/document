# 机器部署

```bash
192.168.1.71 saas01
192.168.1.72 saas02
192.168.1.73 saas03
192.168.1.71 saas01.cloud.com  # K8s-master Docker
192.168.1.72 saas02.cloud.com  # K8s-node Docker
192.168.1.73 saas03.cloud.com  # K8s-node Docker
192.168.1.81 proxy.cloud.com # 80: nginx
192.168.1.91 devops.cloud.com  # Jenkins + Maven + Git

# 在devops配置免密登陆
cd ~/.ssh/ & ssh-keygen -t rsa
ssh-copy-id saas01
ssh-copy-id saas02
ssh-copy-id saas03
```

# docker

## docker简介

1. 关键字：集装箱
2. 可移植、轻量级、资源占用少

1. 对于开发人员，Build once run anywhere
2. 对于运维人员，configure run anywhere

## docker安装

```shell
# 1. 卸载老版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 2.设置仓库
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# 3. 使用阿里云
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 4. 安装 Docker Engine-Community
yum install docker-ce docker-ce-cli containerd.io
# 5. 启动docker
systemctl enable docker && systemctl start docker
# 6. 验证docker是否启动
docker run hello-world
```

## docker卸载

```shell
# 删除安装包
yum remove docker-ce
# 删除镜像、容器、配置文件等内容
rm -rf /var/lib/docker
```

## docker指令

```shell
# 查看版本
docker version
# 查看本地镜像列表
docker image ls 或者 docker images
# 拉取镜像eg：nginx
docker pull nginx
# 删除镜像eg：nginx
docker rmi nginx
# 查看容器列表 -a查询所有，不加只查询运行中的
docker ps -a  
# 进入容器-shell指令
docker exec -it [容器id] /bin/bash
# 查看容器日志
docker logs -f [容器id]
# 停止容器
docker stop [容器id]
# 删除容器
docker rm [容器id]

# 启动docker
systemctl enable docker && systemctl start docker

# 查看镜像的详细信息
docker inspect docker.io/mysql:5.7
```

## docker镜像

### mysql

```shell
docker pull mysql:5.6
# -p 端口映射
# -v 存储挂载
# -e 指定环境变量
docker run \
	-p 3306:3306 \
  --name db02 \
  -v /home/mysql/data:/var/lib/mysql \
	-e MYSQL_ROOT_PASSWORD=xiaojianjun \
  -d mysql:5.6
```

## docker部署springboot项目

### 打包springboot为可执行jar

### 编写Dockerfile文件

```sql
# Docker image for springboot file run
# VERSION 0.0.1
# Author: jianjun.xiao
# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER jianjun.xiao <xiaocen655356@163.com>
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为app.jar
ADD shallwe-discovery-server-eureka-1.0.0.jar shallwe-discovery-server-eureka.jar
# 运行jar包
RUN bash -c 'touch /shallwe-discovery-server-eureka.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/shallwe-discovery-server-eureka.jar"]
```

**解释下这个配置文件：**

VOLUME 指定了临时文件目录为/tmp。其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp。该步骤是可选的，如果涉及到文件系统的应用就很有必要了。/tmp目录用来持久化到 Docker 数据文件夹，因为 Spring Boot 使用的内嵌 Tomcat 容器默认使用/tmp作为工作目录
项目的 jar 文件作为 “app.jar” 添加到容器的
ENTRYPOINT 执行项目 app.jar。为了缩短 Tomcat 启动时间，添加一个系统属性指向 “/dev/./urandom” 作为 Entropy Source

如果是第一次打包，它会自动下载java 8的镜像作为基础镜像，以后再制作镜像的时候就不会再下载了。

### 制作镜像

```shell
# 可执行jar要和DockerFile在同级目录下
# -t 参数是指定此镜像的tag名
docker build -t 192.168.1.71:5000/shallwe-eureka .
# 推送到本地镜像仓库
docker push 192.168.1.71:5000/shallwe-eureka:latest
```

### 启动容器

```shell
# -d 参数是让容器后台运行 
# -p 是做端口映射，此时将服务器中的8081端口映射到容器中的8761(项目中端口配置的是8761)端口
docker run -d -p 8081:8761 192.168.1.71:5000/shallwe-eureka
```

# K8s集群搭建

## 初探K8s

### 为什么要用K8s

- **自我修复**：一旦某一个容器崩溃，能够在1秒左右迅速启动新的容器
- **弹性伸缩**：可以根据需要，自动对集群中正在运行的容器数量进行调整

- **服务发现**：服务可以通过自动发现的形式找到它所依赖的服务
- **负载均衡**：如果一个服务启动了多个容器，能够自动实现请求的负载均衡

- **版本回退**：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
- **存储编排**：可以根据容器自身的需求自动创建存储卷

### **K8s 构成组件**



一个完整的 **Kubernetes** 集群是由 **控制节点 master** 、**工作节点 node** 构成的，因此这种集群方式也分为 **一主多从** 和 **多主多从**，而每个节点上又会安装不同组件以提供服务。

#### 

\1. master-控制节点-负责集群的决策（管理）



- **ApiServer** ：资源操作的唯一入口，接收用户输入的命令，提供认证、授权、Api 注册和发现等机制
- **Scheduler**：负责集群资源调度，按照预定的调度策略将 **pod** 调度到相应的 **node** 节点上

- **ControllerManager**：负责维护集群的状态，比如程序部署安排，故障检测，自动扩展，滚动更新等
- **Etcd**：负责存储集群中各种资源对象的信息



\2. node-工作节点-负责为容器提供运行环境



- **Kubelet**：负责维护容器的生命周期，即通过控制 docker 来创建、更新、销毁容器
- **KubeProxy**：负责提供集群内部的服务发现和负载均衡

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1627948941842-c576058a-ef3d-4ff2-b5aa-b8509533be34.jpeg)

### K8s工作原理

1. **kubernetes** 启动后，无论是 **master** 节点 亦或者 **node** 节点，都会将自身的信息存储到 **etcd** 数据库中
2. 创建 XXX 服务，首先会将安装请求发送到 **master** 节点上的 **apiServer** 组件中

1. **apiServer** 组件会调用 **scheduler** 组件来决定应该将该服务安装到哪个 **node** 节点上。这个时候就需要用到 **etcd** 数据库了，**scheduler**会从 **etcd** 中读取各个 **node** 节点的信息，然后按照一定的算法进行选择，并将结果告知给 **apiServer**
2. **apiServer** 调用 **controllerManager** 去调度 **node** 节点，并安装 XXX 服务

1. **node** 节点上的 **kubelet** 组件接收到指令后，会通知**docker**，然后由 **docker** 来启动一个 XXX 的**pod**，**pod** 是 **kubernetes** 中的最小操作单元，容器都是跑在 **pod** 中
2. 以上步骤完成后，XXX 服务便运行起来了，如果需要访问 XXX，就需要通过 **kube-proxy** 来对 **pod** 产生访问的代理，这样外部用户就能访问到这个 XXX 服务

## 环境准备

- 同步服务器时间

```shell
yum install ntpdate -y
ntpdate time.windows.com
```

- 禁用selinux

selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题

```shell
sed -i 's/enforcing/disabled/' /etc/selinux/config
```

- 禁用swap分区

swap分区指的是虚拟内存分区，它的作用是在物理内存使用完之后，将磁盘空间虚拟成内存来使用启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明

```shell
# 永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab
# 临时关闭
# swapoff -a
```

- 修改linux的内核参数

我们需要修改**linux**的内核参数，添加网桥过滤和地址转发功能，编辑

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
> EOF

# 重载
sysctl --system
```

## 安装docker

- 获取docker镜像源

```shell
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```

- 卸载原有的docker

```shell
# 删除安装包
yum remove docker-ce
# 删除镜像、容器、配置文件等内容
rm -rf /var/lib/docker
```

- 安装docker-ce

```shell
# 安装指定版本的docker
yum install docker-ce-19.03.13 -y
```

- 添加配置文件

```shell
# Docker在默认情况下使用的Cgroup Driver为cgroupfs，而kubernetes推荐使用systemd来代替cgroupfs
# 配置加速器加速下载
[root@master ~]# vim /etc/docker/daemon.json

{
	"registry-mirrors": ["https://xxxx.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

- 启动docker

```shell
# 设置开机启动并且启动
systemctl enable docker.service && systemctl start docker
# 验证docker安装是否成功
docker --version
# 查看docker运行窗台
systemctl status docker
```

## 安装Docker私库（可以跳过）

```shell
# 修改内核转发
echo "net.ipv4.ip_forward = 1" >>/etc/sysctl.conf  && sysctl -p
# 拉取registry
docker pull registry:latest
# 创建挂载点
mkdir -p /opt/data/registry
# 运行
docker run -itd -p 5000:5000 \
  --restart always \
  -v /opt/data/registry/:/var/lib/registry \
  --name registry registry:latest
  
# 告诉主机私有仓库的地址
sed -i "s#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry 192.168.1.71:5000#g" /usr/lib/systemd/system/docker.service
systemctl daemon-reload && systemctl restart docker && docker info
# 打标签
docker tag shallwe-eruka:latest 192.168.1.71:5000/shallwe-eureka:latest
# 上传镜像
docker push 192.168.1.71:5000/shallwe-eureka:latest
# 其他节点下载
docker pull 192.168.1.71:5000/shallwe-eureka:latest
```

## 集群初始化

- 由于 **kubernetes** 的镜像源在国外，速度比较慢，因此我们需要切换成国内的镜像源

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

- 安装**kubeadm**、**kubelet**和**kubectl** 三个组件

```shell
yum install kubelet-1.19.4 kubeadm-1.19.4 kubectl-1.19.4 -y
systemctl enable kubelet.service
# 查看安装情况
yum list installed | grep kubelet
yum list installed | grep kubeadm
yum list installed | grep kubectl
# 查看安装的版本
kubelet --version
```

- master节点初始化集群（在master节点执行）

```shell
# 配置主机host
# CPU要大于2核
# 内存要大于2G

# 创建集群
# 由于默认拉取镜像地址 k8s.gcr.io 国内无法访问，这里指定阿里云镜像仓库地址
# 下面的ip地址192.168.1.71为master机器的地址
[root@saas01 opt]# kubeadm init \
--apiserver-advertise-address=192.168.1.71 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.19.4 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16


# 可能会出现的问题
# Kubernetes version: v1.21.1，依赖/coredns/coredns:v1.8.0，registry.cn-hangzhou.aliyuncs.com/google_containers/coredns/coredns:v1.8.0 不存在。
docker pull registry.aliyuncs.com/google_containers/coredns:1.8.0
docker tag registry.aliyuncs.com/google_containers/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns:v1.8.0
docker rmi registry.aliyuncs.com/google_containers/coredns:1.8.0

# 集群初始化成功之后master执行(会有提示的)
[root@saas01 opt]# mkdir -p $HOME/.kube
[root@saas01 opt]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@saas01 opt]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 查看节点信息
[root@saas01 opt]# kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
saas01   NotReady   master   87s   v1.19.4
```

- node节点加入集群（在node节点执行）

```shell
[root@saas02 opt]# kubeadm join 192.168.1.71:6443 --token 94cbmb.ndsw4vw28gvh0r94 \
    --discovery-token-ca-cert-hash sha256:d51833ddc61eeb2d2a6f280d7011cb2e5b65d620bf0dca5fc4a996506df019b4
    
# 可能会出现ipv4的问题
sysctl -w net.ipv4.ip_forward=1
```

- master节点获取节点信息

```shell
# 但是我们这个时候查看集群状态都是为NotReady，这是因为还没有配置网络插件
kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
saas01   NotReady   master   87s   v1.19.4
saas02   NotReady   <none>   87s   v1.19.4
saas03   NotReady   <none>   87s   v1.19.4
```

- 安装网络插件**flannel（在master上执行）**

下载**flannel配置文件**

```shell
# 可能会下载不下来，github下载之后本地上传
wget https://github.com/flannel-io/flannel/tree/master/Documentation/kube-flannel.yml
```

上传安装包至master服务器执行以下指令

```shell
# 使用配置文件启动fannel,配置文件详解见下文
[root@saas01 ~]# kubectl apply -f kube-flannel.yml
# 再次查看集群节点的状态
[root@saas01 ~]# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
saas01   Ready    master   7h42m   v1.19.4
saas02   Ready    <none>   7h25m   v1.19.4
saas03   Ready    <none>   7h24m   v1.19.4

# 查看运行时容器pod （一个pod里面运行了多个docker容器）
[root@saas01 ~]# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-6d56c8448f-7qzbg         1/1     Running   0          7h45m
coredns-6d56c8448f-8dkkp         1/1     Running   0          7h45m
etcd-saas01                      1/1     Running   1          7h45m
kube-apiserver-saas01            1/1     Running   1          7h45m
kube-controller-manager-saas01   1/1     Running   1          7h45m
kube-flannel-ds-fb87m            1/1     Running   0          7h11m
kube-flannel-ds-jr9pw            1/1     Running   0          7h11m
kube-flannel-ds-kw78f            1/1     Running   0          7h11m
kube-proxy-6fq77                 1/1     Running   1          7h45m
kube-proxy-hb4pz                 1/1     Running   0          7h28m
kube-proxy-qtx74                 1/1     Running   0          7h27m
kube-scheduler-saas01            1/1     Running   1          7h45m

# 从上面的信息就可以大致看出 k8s 的架构
```

这个时候所有节点的状态都是Ready 的状态，到此为止，我们的 **k8s** 集群就算搭建完成了！

## 集群功能验证

- 创建一个deployment

```shell
[root@master ~]# kubectl create deployment shallwe-eruka --image=shallwe-eruka
deployment.apps/shallwe-eruka created

[root@master ~]# kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           31s
```

- 创建一个service提供访问

```shell
[root@master ~]# kubectl expose deploy shallwe-eruka --port=8761 --target-port8761 --type=NodePort
service/shallwe-eruka exposed

[root@master ~]# kubectl get svc 
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx        NodePort    10.110.224.214   <none>        80:31771/TCP   5s
```

## K8s配置文件详解（https://www.kubernetes.org.cn/1414.html）

kubectl apply -f xxxx-xxxx.yml

xxxx-xxxx.yml

```shell
apiVersion: v1 # 【必须】版本号
kind: Pod # 【必选】Pod
metadata: # 【必选-Object】元数据
  name: String # 【必选】 Pod的名称
  namespace: String # 【必选】 Pod所属的命名空间
  labels: # 【List】 自定义标签列表
    - name: String
  annotations: # 【List】 自定义注解列表
    - name: String
spec: # 【必选-Object】 Pod中容器的详细定义
  containers: # 【必选-List】 Pod中容器的详细定义
    - name: String # 【必选】 容器的名称
      image: String # 【必选】 容器的镜像名称
      imagePullPolicy: [Always | Never | IfNotPresent] # 【String】 每次都尝试重新拉取镜像 | 仅使用本地镜像 | 如果本地有镜像则使用，没有则拉取
      command: [String] # 【List】 容器的启动命令列表，如果不指定，则使用镜像打包时使用的启动命令
      args: [String] # 【List】 容器的启动命令参数列表
      workingDir: String # 容器的工作目录
      volumeMounts: # 【List】 挂载到容器内部的存储卷配置
        - name: String # 引用Pod定义的共享存储卷的名称，需使用volumes[]部分定义的共享存储卷名称
          mountPath: Sting # 存储卷在容器内mount的绝对路径，应少于512个字符
          readOnly: Boolean # 是否为只读模式，默认为读写模式
      ports: # 【List】 容器需要暴露的端口号列表
        - name: String  # 端口的名称
          containerPort: Int # 容器需要监听的端口号
          hostPort: Int # 容器所在主机需要监听的端口号，默认与containerPort相同。设置hostPort时，同一台宿主机将无法启动该容器的第二份副本
          protocol: String # 端口协议，支持TCP和UDP，默认值为TCP
      env: # 【List】 容器运行前需设置的环境变量列表
        - name: String # 环境变量的名称
          value: String # 环境变量的值
      resources: # 【Object】 资源限制和资源请求的设置
        limits: # 【Object】 资源限制的设置
          cpu: String # CPU限制，单位为core数，将用于docker run --cpu-shares参数
          memory: String # 内存限制，单位可以为MB，GB等，将用于docker run --memory参数
        requests: # 【Object】 资源限制的设置
          cpu: String # cpu请求，单位为core数，容器启动的初始可用数量
          memory: String # 内存请求，单位可以为MB，GB等，容器启动的初始可用数量
      livenessProbe: # 【Object】 对Pod内各容器健康检查的设置，当探测无响应几次之后，系统将自动重启该容器。可以设置的方法包括：exec、httpGet和tcpSocket。对一个容器只需要设置一种健康检查的方法
        exec: # 【Object】 对Pod内各容器健康检查的设置，exec方式
          command: [String] # exec方式需要指定的命令或者脚本
        httpGet: # 【Object】 对Pod内各容器健康检查的设置，HTTGet方式。需要指定path、port
          path: String
          port: Number
          host: String
          scheme: String
          httpHeaders:
            - name: String
              value: String
        tcpSocket: # 【Object】 对Pod内各容器健康检查的设置，tcpSocket方式
          port: Number
        initialDelaySeconds: Number # 容器启动完成后首次探测的时间，单位为s
        timeoutSeconds: Number  # 对容器健康检查的探测等待响应的超时时间设置，单位为s，默认值为1s。若超过该超时时间设置，则将认为该容器不健康，会重启该容器。
        periodSeconds: Number # 对容器健康检查的定期探测时间设置，单位为s，默认10s探测一次
        successThreshold: 0
        failureThreshold: 0
      securityContext:
        privileged: Boolean
  restartPolicy: [Always | Never | OnFailure] # Pod的重启策略 一旦终止运行，都将重启 | 终止后kubelet将报告给master，不会重启 | 只有Pod以非零退出码终止时，kubelet才会重启该容器。如果容器正常终止（退出码为0），则不会重启。
  nodeSelector: object # 设置Node的Label，以key:value格式指定，Pod将被调度到具有这些Label的Node上
  imagePullSecrets: # 【Object】 pull镜像时使用的Secret名称，以name:secretkey格式指定
    - name: String
  hostNetwork: Boolean # 是否使用主机网络模式，默认值为false。设置为true表示容器使用宿主机网络，不再使用docker网桥，该Pod将无法在同一台宿主机上启动第二个副本
  volumes: # 【List】 在该Pod上定义的共享存储卷列表
    - name: String # 共享存储卷的名称，volume的类型有很多emptyDir，hostPath，secret，nfs，glusterfs，cephfs，configMap
      emptyDir: {} # 【Object】 类型为emptyDir的存储卷，表示与Pod同生命周期的一个临时目录，其值为一个空对象：emptyDir: {}
      hostPath: # 【Object】 类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: String # Pod所在主机的目录，将被用于容器中mount的目录
      secret: # 【Object】类型为secret的存储卷，表示挂载集群预定义的secret对象到容器内部
        secretName: String
        items:
          - key: String
            path: String
      configMap: # 【Object】 类型为configMap的存储卷，表示挂载集群预定义的configMap对象到容器内部
        name: String
        items:
          - key: String
            path: String
```

## K8s指令

```shell
kubectl get nodes # 获取所有的节点
kubectl get pods --all-namespaces -o wide # 获取所有命名空间的pod资源
kubectl get deployments --all-namespaces  # 获取所有命名空间的deployment资源
kubectl get services --all-namespaces # 获取所有命名空间得额service资源

kubectl apply -f xxxx-xxxx.yaml # 创建资源
kubectl delete -f xxxx-xxxx.yaml # 删除资源

kubectl describe pod [pad-name] -n [namespace] # 查看namespace下的pod的详细信息
kubectl delete pod [pad-name] -n [namespace] # 删除namespace下的pod资源

kubectl get ns # 查看所有的namespace
kubectl create ns shallwe # 创建名称为shallwe的namespace
```

# Jenkins安装

## 持续集成的过程

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1627776788996-a358b2b1-20f8-49bd-b62a-a43abfe7bce0.png)

## 环境准备

- JDK
- Git

```shell
yum -y install git
# 查看git path
which git

/usr/bin/git
```

- Maven

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
# 解压
# 配置环境变量
vim /etc/profile
# 重载配置
source /etc/profile
```

## 安装Jenkins

- 下载程序包并启动Jenkins

```shell
wget https://get.jenkins.io/war-stable/2.289.3/jenkins.war
# 启动，不指定端口默认为8080
nohup java -jar jenkins.war --httpPort=9000 &
# 关闭
curl http://devops.cloud.com:9000/exit
# 重启
curl http://devops.cloud.com:9000/restart
# 重载配置文件
curl http://devops.cloud.com:9000/reload
```

- 访问localhost[:8080](http://docker.cloud.com:8080)进去安装引导

```shell
# 密码
cat /root/.jenkins/secrets/initialAdminPassword
```

- 安装插件

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1627813000881-58437bb9-863a-474b-affa-70f9e5985482.png)

- 创建用户
- 安装Maven插件

```shell
Manage Jenkins -> Manage Plugins -> 可选插件 -> Maven Integration plugin 安装 -> 重启Jenkins
curl http://devops.cloud.com:9000/restart
```

- 配置JDK、Git、Maven

Manage Jenkins -> Global Tool Configuration

## 构建项目

### 设置jenkins github webhook

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628008721346-5f026c00-8c17-4ab9-87f2-36be6bcc30a5.png)![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628008757365-bb318e32-5a59-4b99-b117-ae94348714a2.png)