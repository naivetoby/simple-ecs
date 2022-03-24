#### 安装 GitLab

~~~bash
cd ~/gitlab
vim ~/gitlab/docker-compose.yml
------------------
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:13.12.8-ce.0
    container_name: gitlab
    restart: always
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://git.toby.vip'
    ports:
      - '443:443'
      - '80:80'
      - '22:22'
      - '5050:5050'
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab
      - /backup:/backup
    logging:
      driver: json-file
      options:
        max-size: 100m

  gitlab-runner:
    image: gitlab/gitlab-runner
    container_name: gitlab-runner
    restart: always
    depends_on:
      - gitlab
    volumes:
      - ./gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
    logging:
      driver: json-file
      options:
        max-size: 100m
    extra_hosts:
      - "git.toby.vip:172.16.0.99"
------------------
docker-compose up -d

vim ~/gitlab/gitlab/config/gitlab.rb
------------------
### Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'git@toby.vip'
gitlab_rails['gitlab_email_display_name'] = 'Toby GitLib'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "~"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "git@toby.vip"
gitlab_rails['smtp_password'] = "***"
gitlab_rails['smtp_domain'] = "~"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

gitlab_rails['backup_path'] = "/backup"
gitlab_rails['backup_keep_time'] = 604800  

registry['storage_delete_enabled'] = true
------------------

# 开启 gitlab 自动备份
crontab -e
0 5 * * * /root/gitlab/crontab_gitlab.sh clear
0 6 * * * /root/gitlab/crontab_gitlab.sh backup

vim /root/gitlab/crontab_gitlab.sh
------------------
#！ /bin/bash
case "$1" in
    clear)
            docker exec gitlab gitlab-ctl registry-garbage-collect -m
            ;;
    backup)
            docker exec gitlab gitlab-rake gitlab:backup:create
            ;;
esac

chmod +x /root/gitlab/crontab_gitlab.sh
------------------

# 注册 gitlab-runner
docker exec -it gitlab-runner gitlab-ci-multi-runner register
https://git.toby.vip/
xxxx
docker-runner
docker-runner
docker
docker:stable


vim gitlab-runner/config/config.toml
------------------
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "docker-runner"
  url = "https://git.toby.vip/"
  token = "~"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:stable"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/certs/client", "/var/run/docker.sock:/var/run/docker.sock"]
    extra_hosts = ["git.toby.vip:172.16.0.99"]
    pull_policy = ["if-not-present"]
    shm_size = 0

[[runners]]
  name = "android-runner"
  url = "https://git.toby.vip/"
  token = "~"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "git.toby.vip:5050/docker-image/android:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0

[[runners]]
  name = "java-runner"
  url = "https://git.toby.vip/"
  token = "~"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "git.toby.vip:5050/docker-image/maven:latest"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/certs/client", "/var/run/docker.sock:/var/run/docker.sock"]
    extra_hosts = ["git.toby.vip:172.16.0.99"]
    pull_policy = ["if-not-present"]
    shm_size = 0

[[runners]]
  name = "node-runner"
  url = "https://git.toby.vip/"
  token = "~"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:stable"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/certs/client", "/var/run/docker.sock:/var/run/docker.sock"]
    extra_hosts = ["git.toby.vip:172.16.0.99"]
    pull_policy = ["if-not-present"]
    shm_size = 0
------------------

# 优化docker仓库上传速度(在宿主机上编辑内网IP并登录账号)

vim /etc/hosts
x.x.x.x(内网IP) git.toby.vip

docker login git.toby.vip:5050
git@toby.vip
个人访问令牌(api)
~~~
