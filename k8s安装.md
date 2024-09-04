## 安装docker

1. 准备三台虚拟机，操作系统为 `Ubuntu 22.04.3 LTS`，分别修改主机名为 node1，node2，node3

   ~~~shell
   hostnamectl set-hostname node1
   ~~~

   修改 `/etc/hosts` 文件

   ~~~
   192.168.56.2 node1
   192.168.56.3 node2
   192.168.56.4 node3
   ~~~

2. ubuntu 使用 `timesyncd` 进行时钟同步，修改时区并配置时钟同步服务器为 `ntp.ntsc.ac.cn`

   修改 `/etc/systemd/timesyncd.conf`，新增配置 `NTP=ntp.ntsc.ac.cn`

   ```shell
   [Time]
   NTP=ntp.ntsc.ac.cn
   #FallbackNTP=ntp.ubuntu.com
   #RootDistanceMaxSec=5
   #PollIntervalMinSec=32
   #PollIntervalMaxSec=2048
   ```

   重启服务生效

   ~~~shell
   systemctl restart systemd-timesyncd
   ~~~

   修改时区

   ```shell
   timedatectl set-timezone Asia/Shanghai
   ```

3. 安装 docker

   1. 更新操作系统中的软件版本

      ~~~shell
      apt update
      apt upgrade
      ~~~

   2. 添加docker官方GPG密钥

      ~~~shell
      curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
      ~~~

   3. 添加docker软件源

      ~~~shell
      add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
      ~~~

   4. 安装docker

      ~~~shell
      apt install docker-ce docker-ce-cli containerd.io
      ~~~

   5. 默认情况下，只有root用户和docker组的用户才能运行docker命令。我们可以将当前用户添加到docker组，以避免每次使用docker时都需要使用sudo

      ~~~shell
      sudo usermod -aG docker $USER
      ~~~

   6. 安装完成后，docker会自动启动

      ~~~shell
      systemctl status docker.service 
      ● docker.service - docker Application Container Engine
           Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
           Active: active (running) since Tue 2023-12-05 14:56:34 CST; 6min ago
      TriggeredBy: ● docker.socket
             Docs: https://docs.docker.com
         Main PID: 2842 (dockerd)
            Tasks: 11
           Memory: 34.7M
              CPU: 602ms
           CGroup: /system.slice/docker.service
                   └─2842 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
      ~~~
   
   7. 查看docker版本

      ~~~shell
      $ docker version
       Version:           27.2.0
       API version:       1.47
       Go version:        go1.21.13
       Git commit:        3ab4256
       Built:             Tue Aug 27 14:15:13 2024
       OS/Arch:           linux/amd64
       Context:           default
      
      Server: Docker Engine - Community
       Engine:
        Version:          27.2.0
        API version:      1.47 (minimum version 1.24)
        Go version:       go1.21.13
        Git commit:       3ab5c7d
        Built:            Tue Aug 27 14:15:13 2024
        OS/Arch:          linux/amd64
        Experimental:     false
       containerd:
        Version:          1.7.21
        GitCommit:        472731909fa34bd7bc9c087e4c27943f9835f111
       runc:
        Version:          1.1.13
        GitCommit:        v1.1.13-0-g58aa920
       docker-init:
        Version:          0.19.0
        GitCommit:        de40ad0
      ~~~
      
   8. 配置镜像加速
   
      在 `/etc/docker/daemon.json` 中新增以下内容
   
      ~~~
      {
                 "registry-mirrors": ["https://docker.m.daocloud.io","https://dockerproxy.com","https://docker.mirrors.ustc.edu.cn","https://docker.nju.edu.cn"]
      }
      ~~~
   
      重启 docker 服务 `systemctl restart docker`
   
   9. 运行hello-world镜像
   
      ~~~shell
      $ docker run hello-world
      Unable to find image 'hello-world:latest' locally
      latest: Pulling from library/hello-world
      719385e32844: Pull complete 
      Digest: sha256:c79d06dfdfd3d3eb04cafd0dc2bacab0992ebc243e083cabe208bac4dd7759e0
      Status: Downloaded newer image for hello-world:latest
      
      Hello from Docker!
      This message shows that your installation appears to be working correctly.
      
      To generate this message, Docker took the following steps:
       1. The Docker client contacted the Docker daemon.
       2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
          (amd64)
       3. The Docker daemon created a new container from that image which runs the
          executable that produces the output you are currently reading.
       4. The Docker daemon streamed that output to the Docker client, which sent it
          to your terminal.
      
      To try something more ambitious, you can run an Ubuntu container with:
       $ docker run -it ubuntu bash
      
      Share images, automate workflows, and more with a free Docker ID:
       https://hub.docker.com/
      
      For more examples and ideas, visit:
       https://docs.docker.com/get-started/
      ~~~
   
      查看当前正在运行的container以及镜像
   
      ~~~~shell
      $ docker ps -a
      CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
      9b01647548ea   hello-world   "/hello"   24 seconds ago   Exited (0) 23 seconds ago             elated_borg
      
      $ docker images
      REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
      hello-world   latest    9c7a54a9a43c   7 months ago   13.3kB
      ~~~~

## 安装k8s

1. 修改内核参数以及模块配置（参数设置以及模块装载情况可能已经符合要求，可以视情况决定是否执行）

   ~~~shell
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   overlay
   br_netfilter
   EOF
   
   sudo modprobe overlay
   sudo modprobe br_netfilter
   
   # sysctl params required by setup, params persist across reboots
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-iptables  = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward                 = 1
   EOF
   
   # Apply sysctl params without reboot
   sudo sysctl --system
   ~~~

2. 检查 mac 地址以及 product_uuid，确保每个节点都不相同

   ~~~shell
   $ip addr
   $cat /sys/class/dmi/id/product_uuid
   ~~~

3. 安装以下软件包

   ~~~shell
   sudo apt install -y apt-transport-https ca-certificates curl gpg
   ~~~

4. 下载 Kubernetes 软件包仓库的公共签名密钥

   ~~~shell
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   ~~~

5. 添加合适的 k8s apt 仓库

   ~~~shell
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   ~~~

6. 更新apt包索引，安装 `kubelet`、`kubeadm` 和 `kubectl`，并且保持版本以避免意外升级

   ~~~shell
   sudo apt update
   sudo apt install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ~~~

7. 查看版本

   ~~~
   root@node3:/etc/apt/keyrings# kubeadm version
   kubeadm version: &version.Info{Major:"1", Minor:"28", GitVersion:"v1.28.4", GitCommit:"bae2c62678db2b5053817bc97181fcc2e8388103", GitTreeState:"clean", BuildDate:"2023-11-15T16:56:18Z", GoVersion:"go1.20.11", Compiler:"gc", Platform:"linux/amd64"}
   root@node3:/etc/apt/keyrings# kubectl version
   Client Version: v1.28.4
   Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
   The connection to the server localhost:8080 was refused - did you specify the right host or port?
   root@node3:/etc/apt/keyrings# kubelet --version
   Kubernetes v1.28.4
   ~~~

8. 修改 container 配置

   初始化配置文件并修改

   ~~~shell
   containerd config default > /etc/containerd/config.toml
   #执行
   sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
   sudo sed -i 's@sandbox_image = ".*"@sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.10"@' /etc/containerd/config.toml
   ~~~
   
   重启 containerd
   
   ~~~shell
   systemctl restart containerd.service
   ~~~
   
9. 禁用swap分区

   ~~~shell
   # 临时禁用swap分区
   swapoff -a
   
   # 永久禁用swap分区
   vi /etc/fstab 
   # 注释掉下面的设置
   /swap.img      none    swap    sw      0       0
   ~~~

10. 修改容器运行时配置

    ~~~shell
    crictl config runtime-endpoint unix:///run/containerd/containerd.sock
    ~~~

## 集群初始化

1. 以node1作为master节点，在master节点上执行

   1. 拉取k8s所需要的镜像

      ~~~shell
      kubeadm config images pull  --image-repository registry.aliyuncs.com/google_containers
      ~~~

   2. 初始化master节点

      ~~~shell
      kubeadm init \
        --apiserver-advertise-address=192.168.56.2 \
        --image-repository registry.aliyuncs.com/google_containers \
        --kubernetes-version v1.31.0 \
        --service-cidr=10.96.0.0/12 \
        --pod-network-cidr=10.244.0.0/16 \
        --ignore-preflight-errors=all
      
      # –apiserver-advertise-address #集群通告地址
      # –image-repository #指定阿里云镜像仓库地址
      # –kubernetes-version #K8s版本，与上面安装的一致
      # –service-cidr #集群内部虚拟网络，Pod统一访问入口
      # –pod-network-cidr #Pod网络，与下面部署的CNI网络组件yaml中保持一致，可以不用更改，直接用上面的参数
      # --ignore-preflight-errors #忽略检查错误
      ~~~

      安装成功，获取如下信息

      ~~~
      Your Kubernetes control-plane has initialized successfully!
      
      To start using your cluster, you need to run the following as a regular user:
      
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
      
      Alternatively, if you are the root user, you can run:
      
        export KUBECONFIG=/etc/kubernetes/admin.conf
      
      You should now deploy a pod network to the cluster.
      Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
        https://kubernetes.io/docs/concepts/cluster-administration/addons/
      
      Then you can join any number of worker nodes by running the following on each as root:
      
      kubeadm join 192.168.56.2:6443 --token sl1hlj.ctd00d2todbah9df --discovery-token-ca-cert-hash sha256:dec59c180dc9238ce747f4e3360c4d871c820c4eb99c8a699fface1a5d8b4e20
      ~~~

      join 命令的token只有24小时的有效期，可以通过 `kubeadm token create --print-join-command` 获取新的token

      按照提示，非root用户执行

      ~~~shell
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      ~~~

      root用户在 `~/.bashrc` 中添加以下内容

      ~~~~shell
      export KUBECONFIG=/etc/kubernetes/kubelet.conf
      ~~~~

      执行 `echo 'source <(kubectl completion bash)' >>~/.bashrc` 可以实现k8s命令自动补全（需要自动补全的用户执行），同时执行以下命令可以为 `kubectl` 设置别名

      ~~~shell
      echo 'alias kc=kubectl' >>~/.bashrc
      echo 'complete -o default -F __start_kubectl kc' >>~/.bashrc
      ~~~

   3. 设定容器网络

      首先安装cni插件（所有节点都执行）

      ~~~shell
      #下载并安装cni插件
      wget https://github.com/containernetworking/plugins/releases/download/v1.5.1/cni-plugins-linux-amd64-v1.5.1.tgz
      #新建cni目录并解压
      mkdir cni
      tar -zxvf cni-plugins-linux-amd64-v1.5.1.tgz -C cni
      #将解压得到的所有内容复制到/usr/lib/cni下
      cp -r cni /usr/lib
      ~~~

      使用 flannel 网络，下载 `kube-flannel.yml`

      ~~~shell
      wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
      ~~~

      修改配置与 kubeadm init 中的参数一致

      ~~~
        net-conf.json: |
          {
            "Network": "10.244.0.0/16",
            "Backend": {
              "Type": "vxlan"
            }
          }
      ~~~

      执行 `kubectl apply -f kube-flannel.yml`

      执行 `kubectl get pod --all-namespaces` 发现 `kube-flannel` 报错 `Init:ImagePullBackOff`，需要手动通过代理下载镜像

      ~~~shell
      #查看需要的flannel镜像以及版本
      grep image kube-flannel.yml 
      image: docker.io/flannel/flannel-cni-plugin:v1.5.1-flannel2
      image: docker.io/flannel/flannel:v0.25.6
      image: docker.io/flannel/flannel:v0.25.6
      #使用代理手工拉取
      crictl pull docker.m.daocloud.io/flannel/flannel-cni-plugin:v1.5.1-flannel2
      crictl pull docker.m.daocloud.io/flannel/flannel:v0.25.6
      ~~~

      再次执行 `kubectl get pod --all-namespaces` ，所有pod均正常

      ~~~~shell
      NAMESPACE      NAME                            READY   STATUS    RESTARTS   AGE
      kube-flannel   kube-flannel-ds-sc44r           1/1     Running   0          16m
      kube-system    coredns-66f779496c-d8nq4        1/1     Running   0          31m
      kube-system    coredns-66f779496c-m552w        1/1     Running   0          31m
      kube-system    etcd-node1                      1/1     Running   8          31m
      kube-system    kube-apiserver-node1            1/1     Running   7          31m
      kube-system    kube-controller-manager-node1   1/1     Running   8          31m
      kube-system    kube-proxy-m65cb                1/1     Running   0          31m
      kube-system    kube-scheduler-node1            1/1     Running   8          31m
      ~~~~

      执行 `kubectl get node` 获取主节点状态

      ~~~shell
      NAME    STATUS     ROLES           AGE     VERSION
      node1   Ready      control-plane   44h     v1.31.0
      ~~~

2. 添加子节点

   利用第一步得到的 `kubeadm join` 命令分别在node2、ndoe3 上执行，加入k8s集群
   
   ~~~
   This node has joined the cluster:
   * Certificate signing request was sent to apiserver and a response was received.
   * The Kubelet was informed of the new secure connection details.
   
   Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
   ~~~
   
   root用户在 `~/.bashrc` 中添加以下内容
   
   ~~~~shell
   export KUBECONFIG=/etc/kubernetes/kubelet.conf
   ~~~~
   
   非root用户执行
   
   ~~~shell
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/kubelet.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   #子节点需要修改一个文件的权限否则非root用户无法执行kubectl命令
   sudo chmod 604 `ls -l /var/lib/kubelet/pki/kubelet-client-current.pem|xargs -n1|tail -n1`
   ~~~
   
   执行 `echo 'source <(kubectl completion bash)' >>~/.bashrc` 可以实现k8s命令自动补全（需要自动补全的用户执行）
   
   子节点会遇到与master节点相同的 `kube-flannel` 问题，同样需要手动下载镜像
   
   在 node1 上执行 `kubectl get nodes` 查看子节点状态，如果是 `NotReady` 则重启一下 containerd 服务
   
   ~~~shell
   kubectl get node
   NAME    STATUS   ROLES           AGE   VERSION
   node1   Ready    control-plane   60m   v1.28.4
   node2   Ready    <none>          20m   v1.28.4
   node3   Ready    <none>          12m   v1.28.4
   ~~~

## 安装 k8s dashboard

安装官方 k8s UI 管理工具

在 node1 执行

~~~~
#创建资源
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
#检查资源创建情况
kubectl get all -n kubernetes-dashboard
#采用 NodePort 的方式将 dashboard 的端口暴露
kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard
#修改后的内容如下
spec:
  clusterIP: 10.111.179.170
  clusterIPs:
  - 10.111.179.170
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30443
    port: 443
    protocol: TCP
    targetPort: 8443
#查看修改后的服务
kubectl get svc -n kubernetes-dashboard
#查看 dashboard 所在的 node
kubectl get pods -o wide -n kubernetes-dashboard
#根据 node 的 ip 登录 dashboard
https://nodeip:30443
~~~~

上述步骤完成了 dashboard 的安装，接下来需要创建 dashboard 的服务账户

~~~~shell
#创建一个名为 k8s-serviceaccount.yaml 的 yaml 文件，内容如下
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
#执行创建命令
kubectl create -f k8s-serviceaccount.yaml
#检查创建情况
kubectl get sa -n kubernetes-dashboard
#新版本 k8s 不会为 sa 自动创建 secret，需要手动创建，新建一个 sa-secret.yaml 文件并写入以下内容
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-admin-token
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: dashboard-admin
type: kubernetes.io/service-account-token
#执行创建命令
kubectl apply -f sa-secret.yaml
#查看详情
kubectl -n kubernetes-dashboard get secrets
#获取 token
kubectl -n kubernetes-dashboard describe secrets dashboard-admin-token
#将 token 输入 dashboard 登录页面即可登录
~~~~

## 参考文献

[kubernetes(k8s)集群超级详细超全安装部署手册 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/627310856?utm_id=0)

[分区扩容命令_resizepart-CSDN博客](https://blog.csdn.net/qq_39983826/article/details/122339291)

[virtualbox 扩展动态磁盘 Centos7扩容_virtualbox 磁盘扩容_知其黑、受其白的博客-CSDN博客](https://blog.csdn.net/weiguang102/article/details/129612922)

[lvextend 逻辑卷扩容（xfs_growfs、resize2fs配合扩展文件系统）_MssGuo的博客-CSDN博客](https://blog.csdn.net/MssGuo/article/details/120475752)

[Linux磁盘管理：LVM逻辑卷基本概念及LVM的工作原理](https://wangying.sinaapp.com/archives/2301)
