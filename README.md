# 基于K8s+jenkins+gitlab实现devops

## 云原生

什么是云原生？

![image-20240413083807568](E:\Study\gitdemo\jenkins\image\image-20240413083807568.png)

为私有云公有云混合云提供可弹性扩展的环境

云 - cloud 包括(IaaS,PaaS,SaaS)

原生 -- native 项目设计之初要考虑到云环境的适配性，要为云环境而生 native英文指的是土生土长的 原生的。

![image-20240413083839632](E:\Study\gitdemo\jenkins\image\image-20240413083839632.png)

云原生的核心技术:微服务+Devops+CD+容器化

微服务指的是将一个项目划分成很多微小的项目去开发部署上线最后将项目整合起来

Devops指的是持续开发持续集成的开发测试运维一体化协同工作的自动化软件交付模式的落地实践，借助jenkins等自动化集成工具等技术手段实现的解决的沟通问题

CD 持续交付 对于频繁发布的业务能够做多快速交付快速反馈从而降低房补风险

容器化 所有业务和环境都打包成一个镜像，相当于模板，后面如果想使用直接实例化镜像即可。容器有点启动速度快，无OS开销，资源利用率高，可以实现资源隔离(namespace)和资源限制(cgroup)。

![image-20240413083858898](E:\Study\gitdemo\jenkins\image\image-20240413083858898.png)

### 三种类型的云服务

![image-20240413084836981](E:\Study\gitdemo\jenkins\image\image-20240413084836981.png)

**IaaS**

*基础设施及服务* 及云服务商提供给你基础的设施机器硬盘cpu等云设备，其他包括os以及中间件都需要用户自己从零开始手撕。挖地基，搭框架

**PaaS**

*平台及服务* 提供了对开发者较友好的平台，比如安装好了操作系统，mysql数据库，java环境等一些开发软件，类似于毛坯房 刷了墙漆，铺了地板装上了门 装了窗户

**SaaS**

软件及服务 啥都给你配好了 用户或者商户购买此产品后直接就可以使用进行业务 类似于精装修的成品房 拎包入住

![image-20240413085255639](E:\Study\gitdemo\jenkins\image\image-20240413085255639.png)
![image-20240413085322296](E:\Study\gitdemo\jenkins\image\image-20240413085322296.png)

可以看到IaaS就提供给你了服务器，存储和网络然后PaaS在此基础上给你提供了中间件操作系统运行环境等

最后的SaaS在PaaS的基础上提供了可以直接使用的应用

最后我们再介绍一下私有云公有云混合云。

![image-20240413085637865](E:\Study\gitdemo\jenkins\image\image-20240413085637865.png)

看以上这张图即可

我的理解：

私有云:大型企业或者个人搭建的云计算平台供企业内部使用 安全性较高

公有云: 云服务提供商提供的公共的云计算平台，用户企业只要购买云服务提供商的云产品即可直接使用无需自己搭建，减少搭建维护的成本

混合云: 私有云+公有云 优点:灵活性强 可扩展性强

![image-20240413085902990](E:\Study\gitdemo\jenkins\image\image-20240413085902990.png)

在此基础上按照服务层次可以将云计算分为IaaS，PaaS，SaaS

至此我们关于云原生的介绍就到此结束

接下来我们围绕云原生的核心技术展开讲解！

参考文章：

[什么是云原生？| Oracle 中国](https://www.oracle.com/cn/cloud/cloud-native/what-is-cloud-native/)

[什么是云原生？这回终于有人讲明白了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/150190166)

**我们需要用到的技术工具:**

持续继承工具:

- jenkins
- gitlabci
- tekton

## devops CI CD介绍

 Continuous Integration (*CI*)  持续继承

Continuous Delivery (*CD*)   持续交付

一个软件的交付流程

![image-20240412144608813](E:\Study\gitdemo\jenkins\image\image-20240412144608813.png)

一个软件从零开始到最终的部署上线交付大概经历了如上的几个阶段，基于这几个阶段，我们软件交付模式的模型经历了三个阶段：

**阶段一:瀑布式流程**

![image-20240412144747560](E:\Study\gitdemo\jenkins\image\image-20240412144747560.png)

瀑布式开发注重的是软件从设计到开发到测试到部署是有序的执行的，这种模式的缺陷软件开发周期较长，灵活性较差，所以我们后来引入了敏捷开发，这也是很多公司目前使用的软件交付模型

**阶段二:敏捷开发**

敏捷开发注重开发不注重运维，主要做的是讲项目任务分成一小段一小段的去开发 开发完后紧接着立即测试，然后最终将这些开发测试后的小任务拼接成大任务，最后交给运维工程师部署上线

![image-20240412145009400](E:\Study\gitdemo\jenkins\image\image-20240412145009400.png)

**阶段三:devops**

![image-20240412145028614](E:\Study\gitdemo\jenkins\image\image-20240412145028614.png)

一句话概括:devops是一种开发测试运维一体化协同工作的持续交付持续集成的自动化软件交付模式的落地实践。这种落地实践需要技术和工具来解决开发测试和运维之间的沟通成本，所以我们这节课就是这个jenkins流水线工具来做到将devops思想实践化。

devops的核心是**自动化**，靠什么实现自动化呢？其实就是我们的技术和工具

devops的工具链

![image-20240412145256436](E:\Study\gitdemo\jenkins\image\image-20240412145256436.png)

靠这些工具我们的devops才具有了可落地性

## 环境准备

安装k8s集群，准备三台主机

| 主机名     | ip             | 配置参数                                                    |
| ---------- | -------------- | ----------------------------------------------------------- |
| k8s-master | 192.168.208.10 | 2h2g(apiserver,kube-scheduler,kube-controller manager,etcd) |
| k8s-node1  | 192.168.208.10 | 2h4g(kubelet,kube-proxy) 这台机器安装jenkins                |
| k8s-node2  | 192.168.208.30 | 2h6g 资源尽量给多点 这台机器安装gitlab 因为gitlab比较吃内存 |

- 在所有机器上关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

- 在所有机器上关闭selinux

```bash
sed -i 's/enforcing/disabled/' /etc/selinux/config 
setenforce 0
```

- 在所有机器上关闭swap交换分区

```bash
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

- 修改主机名

```bash
hostnamectl set-hostname k8s-master/k8s-node1/k8s-node2
bash
```

- 添加ip与主机名的映射关系

```bash
cat <<EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.208.10 k8s-master
192.168.208.20 k8s-node1
192.168.208.30 k8s-node2
EOF
```

- 将桥接的ipv4流量传递到iptables链

```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

- 为所有节点安装docker

```bash
yum install wget.x86_64 -y
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce-20.10.11 -y
```

### 集群部署

- 为所有节点修改仓库，安装kubeadm，kubelet，kubectl

解释一下这个kubeadm是k8s官方提供的一键部署安装k8s集群的工具，我们之后就用这个工具一键安装k8s集群，kubelet是用来创建销毁pod的，kubectl是客户端命令行工具

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install kubelet-1.22.2 kubeadm-1.22.2 kubectl-1.22.2 -y
systemctl enable kubelet && systemctl start kubelet
```

- 修改docker配置

```bash
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{ 
  "registry-mirrors": ["https://w4m5cbuv.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
  
}
EOF
systemctl daemon-reload 
systemctl restart docker.service 
systemctl restart kubelet.service 
```

- master节点初始化(仅在master节点配置)

```bash
kubeadm init \
--apiserver-advertise-address=192.168.208.10 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.2 \
--control-plane-endpoint k8s-master \
--service-cidr=172.16.0.0/16 \
--pod-network-cidr=10.244.0.0/16
```

![image-20240412151721547](E:\Study\gitdemo\jenkins\image\image-20240412151721547.png)

- 安装他给的提示创建文件授权

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- 复制那一串kubeadm join来到node节点让node节点加入k8s集群

```bash
kubeadm join k8s-master:6443 --token m3tiyw.vrxx7hkbs33gwd9u \
        --discovery-token-ca-cert-hash sha256:d6f0c8db05e038ef1130fa72cc11de88053a7e69886b8984d724c5786fa7401d
```

- 回到master节点敲一下命令

```bash
kubectl get node
# 可以看到都是not ready 是因为我们还没有装网络flannel插件，所有node之间无法通信
```

![image-20240412151804399](E:\Study\gitdemo\jenkins\image\image-20240412151804399.png)

- 安装flannel插件

```bash
vim kube-flannel.yml
---
kind: Namespace
apiVersion: v1
metadata:
  name: kube-flannel
  labels:
    k8s-app: flannel
    pod-security.kubernetes.io/enforce: privileged
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: flannel
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-flannel
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: flannel
  name: flannel
  namespace: kube-flannel
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-flannel
  labels:
    tier: node
    k8s-app: flannel
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "EnableNFTables": false,
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-flannel
  labels:
    tier: node
    app: flannel
    k8s-app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
        image: docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
        image: docker.io/flannel/flannel:v0.25.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: docker.io/flannel/flannel:v0.25.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
kubectl apply -f kube-flannel.yml

# 最好提前拉取一下镜像
docker pull quay.io/coreos/flannel:v0.14.0
```

![image-20240412152201017](E:\Study\gitdemo\jenkins\image\image-20240412152201017.png)

可以看到我的k8s版本是v1.22.2

1.20.0以后k8s的CRI就变成containerd

可以通过`systemctl status containerd`进行查看

![image-20240412152253115](E:\Study\gitdemo\jenkins\image\image-20240412152253115.png)

kubelet就是通过这个containerd的CRI进行创建销毁pod

## 安装jenkins

```bash
mkdir jenkins
kubectl create ns jenkins
vim jenkins/pvc.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-local-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /var/jenkins_home
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - k8s-node2

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-storage

vim jenkins/jenkins.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-local-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /var/jenkins_home
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - k8s-node1

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-storage

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: jenkins
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-master
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      devops: jenkins-master
  template:
    metadata:
      labels:
        devops: jenkins-master
    spec:
    #  nodeSelector:
    #    jenkins: "true"
      serviceAccount: jenkins #Pod 需要使用的服务账号
      initContainers:
        - name: fix-permissions
          image: busybox
          command: ["sh", "-c", "chown -R 1000:1000 /var/jenkins_home"]
          securityContext:
            privileged: true
          volumeMounts:
            - name: jenkinshome
              mountPath: /var/jenkins_home
      containers:
      - name: jenkins
        image: jenkins/jenkins:2.440.2-lts
        imagePullPolicy: IfNotPresent
        ports:
        - name: http #Jenkins Master Web 服务端口
          containerPort: 8080
        - name: slavelistener #Jenkins Master 供未来 Slave 连接的端口
          containerPort: 31000
        volumeMounts:
        - name: jenkinshome
          mountPath: /var/jenkins_home
        env:
        - name: JAVA_OPTS
          value: "-Xms1024m -Xmx4096m -Duser.timezone=Asia/Shanghai -Dhudson.model.DirectoryBrowserSupport.CSP= -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true"
      volumes:
      - name: jenkinshome
        persistentVolumeClaim:
          claimName: jenkins-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins
spec:
  ports:
  - name: http
    port: 8080
    targetPort: 8080
    nodePort: 30080
  - name: slavelistener
    port: 31000
    targetPort: 31000
    nodePort: 31000
  type: NodePort
  selector:
    devops: jenkins-master
    
    
mkdir /var/jenkins_home
chmod 777 -R /var/jenkins_home
kubectl apply -f pvc.yaml
kubectl apply -f jenkins.yaml
```

- 访问

```bash
kubectl get svc -n jenkins
```

![image-20240412172343694](E:\Study\gitdemo\jenkins\image\image-20240412172343694.png)

**查看暴露出来的端口**

浏览器访问输入url `192.168.208.10:30080`

登录密码可以通过`kubectl logs -f jenkins-master-xxx -n jenkins`查看

![image-20240412172528091](E:\Study\gitdemo\jenkins\image\image-20240412172528091.png)

或者来到k8s-node2节点

```bash
cd /var/jenkins_home/secrets
cat initialAdminPassword
```

![image-20240412172611755](E:\Study\gitdemo\jenkins\image\image-20240412172611755.png)

![image-20240412172926279](E:\Study\gitdemo\jenkins\image\image-20240412172926279.png)

![image-20240412172707475](E:\Study\gitdemo\jenkins\image\image-20240412172707475.png)

登录进来选择安装推荐的插件即可

这步安装插件可能需要等上一会因为要连接外网所以速度可能会比较慢

![image-20240412173057406](E:\Study\gitdemo\jenkins\image\image-20240412173057406.png)

如果安装速度很慢网络不行的化那就先跳过这一步，后面我提供另一种方式换国内清华源的方式来下载plugins插件。

![image-20240412174015983](E:\Study\gitdemo\jenkins\image\image-20240412174015983.png)

创建管理员用户

![image-20240412174028512](E:\Study\gitdemo\jenkins\image\image-20240412174028512.png)

实例配置保持默认即可，如果你有域名那就修改一下

![image-20240412174055132](E:\Study\gitdemo\jenkins\image\image-20240412174055132.png)

我这里网速还行就暂时先不加清华源了

支持jenkins持续继承工具就安装好了

接下来我们来到`k8s-node2`在这台机器上我们

**安装gitlab**

gitlab就用docker安装了也比较方便

```bash
docker run -itd --name gitlabce -p 443:443 -p 8076:8076 -p 80:80 --restart always -v /data/devops6/gitlab/config:/etc/gitlab -v /data/devops6/gitlab/logs:/var/log/gitlab -v /data/devops6/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:15.0.3-ce.0
```

**来到宿主机目录修改主机配置文件**

```bash
cd /data/devops/gitlab/config
vim gitlab.rb
# 加上这段配置
external_url 'http://192.168.208.30'
cd /data/gitlab/data/gitlab-rails/etc
vim gitlab.yaml
# 更改主机
host: 192.168.208.30
# 重启gitlab 等等 gitlab因为要初始化所以速度比较慢
# 最后访问192.168.208.30即可进入gitlab
docker restart gitlabce
```

gitlab初始账号:root

gitlab初始密码

```bash
[root@k8s-node2 ~]# cd /data/devops6/gitlab/config/
[root@k8s-node2 config]# cat initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: C+64vZe3fsJt05DEja+eCgmq7uCM//qzs1LVnAPwwfs=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```

浏览器访问`192.168.208.30`

![image-20240412174150740](E:\Study\gitdemo\jenkins\image\image-20240412174150740.png)

输入root和你获取到的密钥即可登录进来了

## 快速入门jenkins实现钉钉机器人告警

思路:我们在gitlab上创建一个仓库，然后把本地的资源提交到仓库上面来，然后在jenkins里面配置流水线找到url和token，然后把jenkins流水线的url和token填入到gitlab仓库的webhook里面实现gitlab和jenkins联动，之后只要我们gitlab这边提交了新内容就会触发jenkins流水线然后执行shell脚本钉钉告警

来到gitlab点击new project然后下一步

![image-20240412174431640](E:\Study\gitdemo\jenkins\image\image-20240412174431640.png)

![image-20240412174457600](E:\Study\gitdemo\jenkins\image\image-20240412174457600.png)

创建项目

来到我们的k8s-node1我们根据他的提示

![image-20240412174528273](E:\Study\gitdemo\jenkins\image\image-20240412174528273.png)

```bash
mkdir -p git/go-demo
cd git/go-demo
git remote add origin http://192.168.208.30/gitlab-instance-4d9b9ba9/go-demo.git
git branch -M main
git push -uf origin main
# 输入你的gitlab的用户名和密码对了这边密码由于是随机的比较麻烦我们可以重置一下gitlab的密码
# 浏览器访问这个网址里面有教程https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password
docker exec -it gitlabce /bin/bash
# 容器内部:gitlab-rake "gitlab:password:reset"
# 账号就输root 密码需要8位数以上 我输的是12345678
docker restart gitlabce 
```

![image-20240412175647595](E:\Study\gitdemo\jenkins\image\image-20240412175647595.png)

好了可以看到我们的项目已经推到了gitlab仓库里面来了

接下来我们配置jenkins

![image-20240412175751473](E:\Study\gitdemo\jenkins\image\image-20240412175751473.png)

manage jenkins --- plugins

![image-20240412175813532](E:\Study\gitdemo\jenkins\image\image-20240412175813532.png)

点击可用插件 安装`gitlab`插件

![image-20240412180034631](E:\Study\gitdemo\jenkins\image\image-20240412180034631.png)

Dashboard > Manage Jenkins > System

配置gitlab地址

![image-20240412180124114](E:\Study\gitdemo\jenkins\image\image-20240412180124114.png)

使用gitlab api token的方式

然后我们来到gitlab申请api token

![image-20240412180242128](E:\Study\gitdemo\jenkins\image\image-20240412180242128.png)

![image-20240412180253120](E:\Study\gitdemo\jenkins\image\image-20240412180253120.png)

会给你一个access token 拿到他来到jenkins

![image-20240412180332654](E:\Study\gitdemo\jenkins\image\image-20240412180332654.png)

填写添加 然后点击`Test connection`测试连接 出现Success即表示连接成功啦

点击新建item 选择自由类型的project

![image-20240412180543796](E:\Study\gitdemo\jenkins\image\image-20240412180543796.png)

![image-20240412180725716](E:\Study\gitdemo\jenkins\image\image-20240412180725716.png)

注意这边的分支是main 因为我们创建gitlab项目的时候分支就是main

![image-20240412180810381](E:\Study\gitdemo\jenkins\image\image-20240412180810381.png)

![image-20240412181006590](E:\Study\gitdemo\jenkins\image\image-20240412181006590.png)

选择构建触发器复制那段url来到我们的gitlab的webhook设置一下让gitlab一旦有提交的事件的时候就会触发这个触发器去实现报警功能

![image-20240412181127324](E:\Study\gitdemo\jenkins\image\image-20240412181127324.png)

点击高级生成secret token然后拿到它复制粘贴到我们的webhook里面来

![image-20240412181217278](E:\Study\gitdemo\jenkins\image\image-20240412181217278.png)

![image-20240412181241340](E:\Study\gitdemo\jenkins\image\image-20240412181241340.png)

然后点击确认会发现没有权限所以提示我们要去先授个权

![image-20240412181331176](E:\Study\gitdemo\jenkins\image\image-20240412181331176.png)

来到我们的go-demo项目点击小扳手

![image-20240412181425478](E:\Study\gitdemo\jenkins\image\image-20240412181425478.png)

![image-20240412181442309](E:\Study\gitdemo\jenkins\image\image-20240412181442309.png)

然后继续来到webhooks界面设置一下即可

![image-20240412181551373](E:\Study\gitdemo\jenkins\image\image-20240412181551373.png)

这个时候我们的webhook就添加成功了

![image-20240412181618413](E:\Study\gitdemo\jenkins\image\image-20240412181618413.png)

来到我们的流水线配置

`Build Steps`选择执行shell脚本

```bash
curl '你的机器人的webhook' \
   -H 'Content-Type: application/json' \
   -d '{"msgtype": "text", 
        "text": {
             "content": "我就是我, 是不一样的烟火"
        }
      }'
```

这个脚本是用来实现钉钉告警的

![image-20240412181802217](E:\Study\gitdemo\jenkins\image\image-20240412181802217.png)

来个钉钉 创一个群然后设置一个机器人复制机器人的webhook

![image-20240412182121566](E:\Study\gitdemo\jenkins\image\image-20240412182121566.png)

流水线运行成功了

![image-20240412182137506](E:\Study\gitdemo\jenkins\image\image-20240412182137506.png)

点击进去可以看到控制台输出状态集等等

```bash
Started by GitLab push by Administrator
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/test
The recommended git tool is: NONE
using credential gitlab_user_pass
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/test/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url http://192.168.208.30/gitlab-instance-4d9b9ba9/go-demo.git # timeout=10
Fetching upstream changes from http://192.168.208.30/gitlab-instance-4d9b9ba9/go-demo.git
 > git --version # timeout=10
 > git --version # 'git version 2.39.2'
using GIT_ASKPASS to set credentials 
 > git fetch --tags --force --progress -- http://192.168.208.30/gitlab-instance-4d9b9ba9/go-demo.git +refs/heads/*:refs/remotes/origin/* # timeout=10
skipping resolution of commit remotes/origin/main, since it originates from another repository
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision f1ea7e724eaee926fdc8d01e9aeeb096b8557148 (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f1ea7e724eaee926fdc8d01e9aeeb096b8557148 # timeout=10
Commit message: "first commit"
First time build. Skipping changelog.
[test] $ /bin/sh -xe /tmp/jenkins8302867855664172248.sh
+ curl https://oapi.dingtalk.com/robot/send?access_token=ee4ee74d092b21126f0ad3e9d567a5797cbedfbca47f9c2d03b02509ae4e9ed8 -H Content-Type: application/json -d {"msgtype": "text", 
        "text": {
             "content": "hahahha我就是我, aaa是不一样的烟火"
        }
      }
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   155  100    27  100   128     87    412 --:--:-- --:--:-- --:--:--   501
{"errcode":0,"errmsg":"ok"}Finished: SUCCESS
```

![image-20240412182216411](E:\Study\gitdemo\jenkins\image\image-20240412182216411.png)

钉钉这边也实现了告警至此我们初步实现了jenkins的一些功能了

manage jenkins --- nodes

![image-20240412182302177](E:\Study\gitdemo\jenkins\image\image-20240412182302177.png)

目前我们就一个节点

这个节点意思就是你的jenkins部署在哪个节点它就会把这个代码下载到哪个节点的哪个文件里面

具体下载到哪个文件可以到可以查看一下控制台输出

![image-20240412182426369](C:\Users\万里\AppData\Roaming\Typora\typora-user-images\image-20240412182426369.png)

这段意思就是他下载到了我们的`k8s-node1`底下`/var/jenkins_home/workspace/test/`下我们可以来到我们的k8s-node1底下查看验证一下

![image-20240412182525274](E:\Study\gitdemo\jenkins\image\image-20240412182525274.png)

确实有我们之前的项目

但是现在有一个问题就是触发流水线工作只有一台机器来进行会出现OSPF问题当并发大的时候吞吐量大的时候一台机器可能顶不住 所以我们可以添加更多的工作node来完成流水线

![image-20240412182657643](E:\Study\gitdemo\jenkins\image\image-20240412182657643.png)

![image-20240412182731076](E:\Study\gitdemo\jenkins\image\image-20240412182731076.png)

启动方式通过Java Web启动代理的方式

![image-20240412182751933](E:\Study\gitdemo\jenkins\image\image-20240412182751933.png)

根据他的提示来到k8s-node2节点上来

先安装java编译器

```bash
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm
yum -y install ./jdk-17_linux-x64_bin.rpm
curl -sO http://192.168.208.10:30080/jnlpJars/agent.jar
nohup java -jar agent.jar -url http://192.168.208.10:30080/ -secret 61fb76f87cd8a4a41c00b0c235a538b3bb78ffed25321959b234fedc18276eb4 -name "k8s-node2" -workDir "/opt/jenkins-home" &
```

