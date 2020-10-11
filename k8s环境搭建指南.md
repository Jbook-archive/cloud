### 1. 环境准备

> windows 10
> 
> vagrant 
>  
> virtualbox 6.1.12-139181
> 
> ubuntu18.04镜像

#### 1.1 vagrant使用指南

1. 下载vagrant程序并安装，[下载地址](https://www.vagrantup.com/downloads.html)，安装后添加环境变量。

2. 下载vagrant对应格式的虚拟机镜像，[下载地址](https://app.vagrantup.com/boxes/search)，使用迅雷下载速度还可以。不过迅雷下载的地址要通过这种方式找到，将地址复制到浏览器中下载后，复制浏览器下载内容中的地址用迅雷下载。
@import "images/image_download.png"

3. 或者使用清华源进行下载`vagrant box add https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cloud-images/bionic/current/bionic-server-cloudimg-amd64-vagrant.box --name ubuntu18`

4. 创建一个新的目录用来存放虚拟机文件。

5. 镜像添加到本地的box中，如`vagrant box add 自定义虚拟机名称 虚拟机url地址或本机路径`如：
```
// 添加镜像
$ vagrant box add ubuntu xenial-server-cloudimg-amd64-vagrant.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'ubuntu' (v0) for provider:
    box: Unpacking necessary files from: file://C:/Users/Administrator/Documents/vagrant/xenial-server-cloudimg-amd64-vagrant.box
    box:
==> box: Successfully added box 'ubuntu' (v0) for 'virtualbox'!

// 查询box是否添加成功
$ vagrant box list
ubuntu (virtualbox, 0)
```

5. 初始化一个基于上述box的虚拟机环境，此时会在目录中生成一个vagrantfile的文件，文件中是启动vagrant的各种配置，如虚拟机的内存和CPU使用等。
```
$ vagrant init ubuntu
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

6. 启动虚拟机，自动将虚拟机的ssh的监听端口22映射到宿主机的2222端口，登录用户名为vagrant，认证方式为private key。
```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: vagrant_default_1599054751499_67292
Vagrant is currently configured to create VirtualBox synced folders with
the `SharedFoldersEnableSymlinksCreate` option enabled. If the Vagrant
guest is not trusted, you may want to disable this option. For more
information on this option, please refer to the VirtualBox manual:

  https://www.virtualbox.org/manual/ch04.html#sharedfolders

This option can be disabled globally with an environment variable:

  VAGRANT_DISABLE_VBOXSYMLINKCREATE=1

or on a per folder basis within the Vagrantfile:

  config.vm.synced_folder '/host/path', '/guest/path', SharedFoldersEnableSymlinksCreate: false
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    ...
==> master: Setting hostname...
==> master: Mounting shared folders...
    master: /vagrant => C:/Users/Administrator/Documents/vagrant
```

7. vagrantfile配置文件，修改配置命令后生效的方法`vagrant reload`重启虚拟机
```
// 要启动的虚拟机名称，对应于vagrant init 名称
config.vm.box = "ubuntu"  

// 调用VBoxManage的modifyvm命令设置CPU内存
config.vm.provider "virtualbox" do |vb|
  v.customize ["modifyvm", :id, "--name", "ubuntu", "--memory", "1024"]
end

// 虚拟机网络
// host-only: 所有虚拟机之间、虚拟机和宿主机均可相互通信，但虚拟机无法使用外网，配置如下：
config.vm.network :private_network, ip: "22.22.22.22"

// bridge: 虚拟机需要设置ip，网络和宿主机是平级的关系，配置待补充。

...

// 设置hostname
config.vm.hostname = "kubernetes"

// 共享目录 第一个参数是主机的目录，第二个参数是虚拟机挂载的目录
config.vm.synced_folder  "/Users/c/data", "/vagrant_data"

// 端口转发，将宿主机的8080转发到虚拟机的80端口
config.vm.network :forwarded_port, guest: 80, host: 8080
```

8. 启动一个集群，待补充


#### 1.2 配置一个ubuntu源
清华大学ubuntu16.04源
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```
清华大学18.04的源
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

阿里18.04的源
```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
```

#### 1.3 docker安装

官方安装教程：https://docs.docker.com/engine/install/ubuntu/

我们使用官方的脚本来安装：
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

安装过程需要一点时间，也可以使用国内的安装脚本
curl -sSL https://get.daocloud.io/docker | sh
```

或者自己新建一个脚本来运行，脚本内容如下：
```
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io
```

在安装的过程中碰到了下载docker报`Hash Sum mismatch`的错误，尝试了多种方法均没有解决。

最后使用了`apt-get install docker.io`来安装老版本的docker。


#### 1.3 k8s的安装

创建一个脚本来安装：
```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated
```

按照上面的脚本安装出现问题，无法安装。决定安装一个minikube来玩玩。

minikube的安装步骤：

1. 安装kubectl
先查看最新的kubectl的版本`https://storage.googleapis.com/kubernetes-release/release/stable.txt`，然后下载kubectl对应的程序
```
sudo wget "https://storage.googleapis.com/kubernetes-release/release/v1.19.0/bin/linux/amd64/kubectl" -O "/usr/local/bin/kubectl"

chmod +x /usr/local/bin/kubectl
```

2. 安装minikube
使用阿里云发布的minikube地址查看最新版本`https://github.com/AliyunContainerService/minikube`，当前最新版本为v1.12.3，下载minikube.
```
sudo curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.12.3/minikube-linux-amd64 && sudo chmod +x minikube && sudo mv minikube /usr/local/bin/
```

3. 启动minikube，切换到root，先安装conntrack，然后再启动。
```
sudo apt-get install -y conntrack

// 启动
minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com

这一步可能会报错，虚拟机内存不足，至少需要2g。
在这个过程中会下载并安装kubelet，kubeadm，kubectl等工具。
```