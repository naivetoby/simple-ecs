#### 安装 RabbitMQ

```bash
vim ~/rabbitmq/Dockerfile
------------------
FROM rabbitmq:3-management
COPY rabbitmq_delayed_message_exchange-3.9.0.ez /plugins
RUN rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange
------------------


vim ~/rabbitmq/docker-compose.yml
------------------
version: '3.8'

services:
  rabbitmq:
    build: .
    container_name: rabbitmq
    hostname: rabbitmq
    restart: always
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - ./data:/var/lib/rabbitmq
    environment:
      - RABBITMQ_DEFAULT_VHOST=/
      - RABBITMQ_DEFAULT_USER={username}
      - RABBITMQ_DEFAULT_PASS={password}
------------------
docker-compose up -d
```
