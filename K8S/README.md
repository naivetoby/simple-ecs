#### 初始化 k8s 宿主机

~~~bash
# 修改服务器主机
hostnamectl set-hostname [k8s-master/k8s-node01/k8s-node02]

# 配置静态 IP
cp /etc/sysconfig/network-scripts/ifcfg-enp0s3 /etc/sysconfig/network-scripts/ifcfg-enp0s3.bak

cat > /etc/sysconfig/network-scripts/ifcfg-enp0s3 << EOF
BOOTPROTO="static"
NAME="enp0s3"
DEVICE="enp0s3"
ONBOOT="yes"
IPADDR=192.168.5.153
NETMASK=255.255.255.0
GATEWAY=192.168.5.1
DNS1=192.168.5.252
DNS2=202.101.172.35
EOF

systemctl restart network

# 配置 HOST
cat >> /etc/hosts << EOF
192.168.5.153   k8s-master
192.168.5.70    k8s-node01
192.168.5.117   k8s-node02
192.168.5.118   k8s-node03
EOF

# 升级系统
yum -y update --exclude=kernel*
# 升级系统内核
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install -y kernel-lt
# 设置开机从新内核启动
grub2-set-default 'CentOS Linux (5.4.172-1.el7.elrepo.x86_64) 7 (Core)'

# 重启
reboot

# 检查是否安装新内核成功
uname -r

# 升级系统
yum -y update --exclude=kernel*

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

# 安装 ntpdate
yum install ntpdate -y
# 执行同步
ntpdate ntp1.aliyun.com
# 写入定时任务
crontab -e
*/5 * * * * ntpdate ntp1.aliyun.com

# 安装管理工具
yum install -y htop

# 密钥登录
vim ~/.ssh/authorized_keys

chmod 700  ~/.ssh
chmod 644  ~/.ssh/authorized_keys

# 新建账号 toby 并赋予管理员权限
useradd toby

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

# 关闭 防火墙
systemctl disable --now firewalld
# 关闭 postfix
systemctl disable --now postfix
# 关闭 NetworkManager
systemctl disable --now NetworkManager
# 关闭 selinux
sed -ri 's/(^SELINUX=).*/\1disabled/' /etc/selinux/config
setenforce 0
# 关闭 swap
sed -ri 's@(^.*swap *swap.*0 0$)@#\1@' /etc/fstab
swapoff -a

# 所有节点修改资源限制
cat > /etc/security/limits.conf <<EOF
*       soft        core        unlimited
*       hard        core        unlimited
*       soft        nproc       1000000
*       hard        nproc       1000000
*       soft        nofile      1000000
*       hard        nofile      1000000
*       soft        memlock     32000
*       hard        memlock     32000
*       soft        msgqueue    8192000
EOF

# 调整内核参数
cat > /etc/sysctl.conf <<EOF
net.ipv4.tcp_keepalive_time=600
net.ipv4.tcp_keepalive_intvl=30
net.ipv4.tcp_keepalive_probes=10
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv4.neigh.default.gc_stale_time=120
net.ipv4.conf.all.rp_filter=0 # 默认为1，系统会严格校验数据包的反向路径，可能导致丢包
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.default.arp_announce=2
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_announce=2
net.ipv4.ip_local_port_range= 45001 65000
net.ipv4.ip_forward=1
net.ipv4.tcp_max_tw_buckets=6000
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_synack_retries=2
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.netfilter.nf_conntrack_max=2310720
net.ipv6.neigh.default.gc_thresh1=8192
net.ipv6.neigh.default.gc_thresh2=32768
net.ipv6.neigh.default.gc_thresh3=65536
net.core.netdev_max_backlog=16384 # 每CPU网络设备积压队列长度
net.core.rmem_max = 16777216 # 所有协议类型读写的缓存区大小
net.core.wmem_max = 16777216
net.ipv4.tcp_max_syn_backlog = 8096 # 第一个积压队列长度
net.core.somaxconn = 32768 # 第二个积压队列长度
fs.inotify.max_user_instances=8192 # 表示每一个real user ID可创建的inotify instatnces的数量上限，默认128.
fs.inotify.max_user_watches=524288 # 同一用户同时可以添加的watch数目，默认8192。
fs.file-max=52706963
fs.nr_open=52706963
kernel.pid_max = 4194303
net.bridge.bridge-nf-call-arptables=1
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
vm.max_map_count = 262144
EOF

# 加载 ipvs 模块
cat >/etc/modules-load.d/ipvs.conf <<EOF
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
EOF
systemctl enable --now systemd-modules-load.service

# 设置 rsyslogd 和 systemd journald
mkdir /var/log/journal 
# 持久化保存日志的目录
mkdir /etc/systemd/journald.conf.d
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent
# 压缩历史日志
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000
# 最大占用空间 1G
SystemMaxUse=1G
# 单日志文件最大 10M
SystemMaxFileSize=10M
# 日志保存时间 2 周
MaxRetentionSec=2week
# 不将日志转发到 syslog
ForwardToSyslog=no
EOF

systemctl restart systemd-journald && systemctl enable systemd-journald

# 安装基础软件
yum -y install curl conntrack ipvsadm iptables ipset jq sysstat libseccomp rsync wget jq psmisc vim net-tools telnet git lrzsz

# 设置防火墙为 iptables
systemctl start iptables && systemctl enable iptables && iptables -F && service iptables save

# 重启
reboot

# 检查 conntrack
lsmod | grep -e ip_vs -e nf_conntrack

# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.12-3.1.el8.x86_64.rpm

yum install docker-ce

systemctl enable --now docker

systemctl is-active docker

# 加速 docker
# Kubernetes 推荐使用 systemd 来代替 cgroupfs
vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://dockerproxy.com",
    "https://xxxx.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
     "max-size": "100m"
   }
}

systemctl daemon-reload && systemctl restart docker

# 安装 Kubeadm（主从配置）
cat > /etc/yum.repos.d/kubernetes.repo <<EOF 
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet-1.23.1-0 kubeadm-1.23.1-0 kubectl-1.23.1-0

systemctl daemon-reload && systemctl restart kubelet && systemctl enable kubelet

# 系统代理
export https_proxy=http://192.168.5.200:7890 http_proxy=http://192.168.5.200:7890 all_proxy=socks5://192.168.5.200:7890

# 创建 docker 代理
mkdir /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http_proxy.conf
---
[Service]
Environment="HTTP_PROXY=http://192.168.5.200:7890"
Environment="HTTPS_PROXY=http://192.168.5.200:7890"
---
systemctl daemon-reload && systemctl restart docker
# 查看 docker 代理是否生效
systemctl show --property=Environment docker

# 下载初始化必备镜像
# kubeadm config images pull
~~~
