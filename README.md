# simple-ecs

服务器基础运维文档

#### 基于 CentOS 7.9 版本

```bash
# 登录服务器
ssh -o ServerAliveInterval=60 root@toby.vip

# 修改服务器主机
hostnamectl set-hostname toby

# 升级系统
yum -y update

# 第三方库
yum install epel-release -y
yum repolist epel
yum repolist epel -v

# 修改字符集及语言
echo "export LANG=en_US.UTF-8" >> /etc/profile
echo "export LANGUAGE=en_US:en" >> /etc/profile

source /etc/profile

# 修改时区
rm -f /etc/localtime \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && rm -f /etc/sysconfig/clock \
    && echo "ZONE=\"Asia/Shanghai\"" >> /etc/sysconfig/clock \
    && echo "UTC=false" >> /etc/sysconfig/clock \
    && echo "ARC=false" >> /etc/sysconfig/clock \
    && echo "SRM=false" >> /etc/sysconfig/clock

# 安装管理工具
yum install -y htop

# 密钥登录
vim ~/.ssh/authorized_keys

chmod 700  ~/.ssh
chmod 644  ~/.ssh/authorized_keys

# 新建账号 toby 并赋予管理员权限
useradd toby

passwd toby

visudo

toby   ALL=(ALL)       NOPASSWD:ALL

mkdir ~/.ssh

vim ~/.ssh/authorized_keys

chmod 700  ~/.ssh
chmod 644  ~/.ssh/authorized_keys

systemctl restart sshd

# 禁止 root 登录及密码登录
vim /etc/ssh/sshd_config

PubkeyAuthentication yes
PermitRootLogin no
PasswordAuthentication no

systemctl restart sshd

# 登录子账号
ssh -o ServerAliveInterval=60 toby@toby.vip

# 授权 docker 命令
sudo usermod -aG docker toby

# 超级管理员权限
sudo -s

# 新建账号 deploy 并赋予管理员权限
useradd deploy

mkdir ~/.ssh

vim ~/.ssh/authorized_keys

chmod 700  ~/.ssh
chmod 644  ~/.ssh/authorized_keys

systemctl restart sshd

# 授权 docker 命令

sudo usermod -aG docker deploy
```

#### 安装 Docker

```bash
# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.5.11-3.1.el7.x86_64.rpm

yum install docker-ce

systemctl enable --now docker

systemctl is-active docker

# 禁用防火墙
systemctl disable firewalld

systemctl status firewalld

# 加速 docker
vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://dockerproxy.com",
    "https://xxxx.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"
  ]
}

# 安装 docker-compose
curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# 开启集群模式
docker swarm init

# 加入集群
docker swarm join --token {token} {ip}:2377

# 创建集群网络
docker network create -d overlay {network-name}

# manager leave
docker swarm leave --force

# worker leave
docker swarm leave
```
