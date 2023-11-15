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

# 将 toby 加入 docker 组
sudo usermod -aG docker toby

# 卸载
sudo dnf -y autoremove htop

```
