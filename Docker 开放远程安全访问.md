#### Docker 开放远程安全访问（开启 2376 端口和 CA 认证）

```bash
# 创建 certs 文件夹，用来存放 CA 私钥和公钥
mkdir -pv /etc/docker/certs
cd /etc/docker/certs

# 创建密码
openssl genrsa -aes256 -out ca-key.pem 4096

# 输入两次密码(随机生成的)
42hirgxxVuWoad-yL8xgiJGv
42hirgxxVuWoad-yL8xgiJGv

# 依次输入密码、国家、省、市、组织名称等
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
{密码}
CN
{省}
{市}
{组织}
{品牌}
{名称}
{邮箱}

# 生成server-key.pem
openssl genrsa -out server-key.pem 4096

# 生成server.csr（把下面的IP换成你自己服务器外网的IP或者域名)
openssl req -subj "/CN={外网IP}" -sha256 -new -key server-key.pem -out server.csr

# 0.0.0.0表示所有ip都可以连接（这里需要注意，虽然 0.0.0.0 可以匹配任意，但是仍需要配置你的外网 ip 和 127.0.0.1，否则客户端会连接不上）
echo subjectAltName = IP:0.0.0.0,IP:{外网IP},IP:127.0.0.1 >> extfile.cnf
# 将Docker守护程序密钥的扩展使用属性设置为仅用于服务器身份验证
echo extendedKeyUsage = serverAuth >> extfile.cnf

# 输入之前设置的密码，生成签名证书
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

# 生成供客户端发起远程访问时使用的 key.pem
openssl genrsa -out key.pem 4096

# 生成client.csr（把下面的 IP 换成你自己服务器外网的 IP 或者域名）
openssl req -subj "/CN=client" -new -key key.pem -out client.csr

# 创建扩展配置文件，把密钥设置为客户端身份验证用
echo extendedKeyUsage = clientAuth > extfile-client.cnf

# 生成 cert.pem，输入前面设置的密码，生成签名证书
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf

# 删除不需要的配置文件和两个证书的签名请求
rm -f client.csr server.csr extfile.cnf extfile-client.cnf

# 为了防止私钥文件被更改以及被其他用户查看，修改其权限为所有者只读
chmod -v 0400 ca-key.pem key.pem server-key.pem

# 为了防止公钥文件被更改，修改其权限为只读
chmod -v 0444 ca.pem server-cert.pem cert.pem

# 修改 Docker 配置，使 Docker 守护程序仅接受来自提供 CA 信任的证书的客户端的连接
vim /lib/systemd/system/docker.service

# 在 ExecStart=/usr/bin/dockerd-current 下面增加
\
-H tcp://0.0.0.0:2376 \
--tlsverify \
--tlscacert=/etc/docker/certs/ca.pem \
--tlscert=/etc/docker/certs/server-cert.pem \
--tlskey=/etc/docker/certs/server-key.pem

# 重新加载daemon并重启docker
systemctl daemon-reload
systemctl restart docker
```