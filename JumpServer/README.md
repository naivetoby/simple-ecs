## Jumpserver Docker-Compose

```sh
docker network create jumpserver

docker-compose -f docker-compose-mysql.yml up -d

docker exec -it jms_mysql /bin/bash
mysql -uroot -p~
create database jumpserver default charset 'utf8mb4';
create user 'jumpserver'@'%' identified by '~';
grant all on jumpserver.* to 'jumpserver'@'%';
flush privileges;

docker-compose -f docker-compose-redis.yml up -d

docker-compose -f docker-compose.yml up -d
```