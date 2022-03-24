#### 安装 Caddy


添加 alidns Dockerfile
```bash
FROM caddy:2.4.6-builder-alpine AS builder

RUN go env -w GOPROXY=https://goproxy.cn

RUN xcaddy build --with github.com/caddy-dns/alidns

FROM caddy:2.4.6-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```


```bash
vim ~/caddy/docker-compose.yml
------------------
version: '3.8'

services:
  caddy:
    image: caddy:2.4.6-alpine
    container_name: caddy
    network_mode: host
    hostname: caddy
    domainname: caddy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/site:/site
      - $PWD/ssl:/ssl
      - data:/data
      - config:/config

volumes:
  data:
  config:
------------------
docker-compose up -d
```

Caddyfile

```bash
import /vhost/*

test.toby.vip {
  encode gzip
  tls /ssl/toby.vip.pem /ssl/toby.vip.key
  reverse_proxy localhost:9000 {
    header_up Host {http.request.host}
    header_up X-Real-IP {http.request.remote.host}
    header_up X-Forwarded-For {http.request.remote.host}
    header_up X-Forwarded-Port {http.request.port}
  }
}

file.toby.vip {
  encode gzip
  tls /ssl/toby.vip.pem /ssl/toby.vip.key
  root * /site/{name}
  file_server
  try_files {path} {path}/ /index.html
}

*.test.toby.vip {
   encode gzip
   tls {
     dns alidns {
       access_key_id {env.ALIYUN_ACCESS_KEY_ID}
       access_key_secret {env.ALIYUN_ACCESS_KEY_SECRET}
     }
   }
   reverse_proxy https://*.test.toby.vip:18443 {
     transport http {
       tls_insecure_skip_verify
     }
     header_up X-Real-IP {http.request.remote.host}
     header_up X-Forwarded-For {http.request.remote.host}
     header_up X-Forwarded-Port {http.request.port}
   }
}
```
