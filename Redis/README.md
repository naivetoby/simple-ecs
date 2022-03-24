#### 安装 Redis

~~~bash
vim ~/redis/docker-compose.yml
------------------
version: '3.8'

services:
  redis:
    image: redis
    container_name: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
      - ./redis.conf:/etc/redis.conf
    command: redis-server /etc/redis.conf
------------------
docker-compose up -d
~~~
