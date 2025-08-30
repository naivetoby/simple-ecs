# 服务器基础运维文档

#### 基于 Rocky Linux 9.3 版本

```bash
# 更换 Rocky Linux 源

# 设置镜像变量
MIRROR=mirrors.aliyun.com/rockylinux
# 执行替换
sudo sed -i.bak \
-e "s|^mirrorlist=|#mirrorlist=|" \
-e "s|^#baseurl=|baseurl=|" \
-e "s|dl.rockylinux.org/\$contentdir|$MIRROR|" \
/etc/yum.repos.d/rocky-*.repo
# 生成缓存
sudo dnf makecache

# 更新系统
sudo dnf -y update

# 更新包数据库
sudo dnf check-update

# 启用 CRB 存储库
sudo dnf config-manager --set-enabled crb

# 安装 EPEL 源
sudo dnf -y install epel-release

# 刷新元数据缓存
sudo dnf makecache

# 修改时区
sudo timedatectl set-timezone Asia/Shanghai

# 安装 Htop
sudo dnf -y install htop

# 安装 Docker
sudo dnf config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl --now enable docker

# Docker 配置
vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://dockerproxy.com",
    "https://xxxx.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"
  ],
  "log-opts": {
     "max-size": "100m"
  },
  "exec-opts": ["native.cgroupdriver=systemd"], # 好像已经默认
  "log-driver": "json-file", # 好像已经默认
  "data-root": "/data/docker" # 修改 Docker 路径, 这是可选的
}

systemctl daemon-reload && systemctl restart docker

# 创建用户 toby
useradd -m toby
# userdel -r toby
su toby
cd ~
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
chmod 700  ~/.ssh
chmod 644  ~/.ssh/authorized_keys

# 免 sudo 权限
visudo
toby   ALL=(ALL)       NOPASSWD:ALL

# 禁止 root 登录及密码登录
vim /etc/ssh/sshd_config

PubkeyAuthentication yes
PermitRootLogin no
PasswordAuthentication no

systemctl restart sshd

# 将 toby 加入 docker 组
sudo usermod -aG docker toby

# 卸载
sudo dnf -y autoremove htop

# 查看一些系统信息
uname -a
cat /etc/rocky-release
locale

```
