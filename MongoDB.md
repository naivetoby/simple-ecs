#### 安装 MongoDB

```bash
vim ~/mongodb/docker-compose.yml
------------------
version: '3.1'

services:
  mongodb:
    image: mongo
    container_name: mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: {username}
      MONGO_INITDB_ROOT_PASSWORD: {password}
------------------
docker-compose up -d
```
