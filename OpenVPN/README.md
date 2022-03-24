#### 安装 OpenVPN

###### 安装 OpenVPN-Server
~~~bash
cd ~/openvpn
vim ~/openvpn/docker-compose.yml
------------------
version: '3.8'

services:
  openvpn:
    image: kylemanna/openvpn
    container_name: openvpn
    cap_add:
     - NET_ADMIN
    ports:
     - "1194:1194/udp"
    restart: unless-stopped
    volumes:
     - ./openvpn-data/conf:/etc/openvpn

docker-compose run --rm openvpn ovpn_genconfig -u udp://vpn.toby.vip

docker-compose run --rm openvpn ovpn_initpki
------------------

Enter New CA Key Passphrase:此处输入密码

Common Name (eg: your user, host, or server name) [Easy-RSA CA] 直接回车

Enter pass phrase for /etc/openvpn/pki/private/ca.key 此处输入上面👆的密码

chown -R $(whoami): ./openvpn-data

docker-compose run --rm openvpn easyrsa build-client-full toby_vpn_client nopass

docker-compose run --rm openvpn ovpn_getclient toby_vpn_client > toby_vpn_client.ovpn

docker-compose up -d
~~~
