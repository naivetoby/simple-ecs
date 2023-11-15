# 服务器基础运维文档

#### 基于 Rocky Linux 9.2 版本

```bash

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
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl --now enable docker

# 创建用户 toby
useradd toby
su tody
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

```
