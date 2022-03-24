#### 安装 Portainer

```bash
vim ~/portainer/docker-compose.yml
------------------
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:2.11.0
    container_name: portainer
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - data:/data
    ports:
     - "8000:8000"
     - "9000:9000"

volumes:
  data:
------------------
docker-compose up -d
```
