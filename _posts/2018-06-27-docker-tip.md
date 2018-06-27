---
layout: post
title: 'Docker Tip 01' 
author: teamsmiley 
date: 2018-06-27
tags: [docker]
image: /files/covers/blog.jpg
category: {program}
---

# Docker 팁 정리 


## 도커의 모든 프로세스는 포그라운드로 돌려야한다.
기본적으로 도커는 프로세스이다. 그래서 프로세스가 멈추면 도커도 자동으로 멈춘다. 
예를 들어 톰캣을 백그라운드로 돌리면 포그라운드가 멈추기 때문에 프로세스자체가 종료되버린다. 

## startup.sh을 도커로 변경해보자. 

```bash
nohup python -O /data/LogImporter/LogImporter22th.py --server game01 &
nohup python -O /data/LogImporter/LogImporter22th.py --server game02 &
nohup python -O /data/LogImporter/LogImporter22th.py --server game03 &
nohup python -O /data/LogImporter/LogImporter22th.py --server game04 &
nohup python -O /data/LogImporter/LogImporter22th.py --server gamewar01 &
nohup python -O /data/LogImporter/LogImporter22th.py --server login &
```

이런 경우에 도커 이미지는 하나이고 아규먼트를 받아서 그 여러개의 프로세스를 만들어야한다.

### 실제 어플리케이션에 하드코딩되있는 아이디 비번을 도커에서 전달받아야한다. 

```python

##### 기존 코드 
#LogDBName = "YOUR DBNAME"
#MysqlHost = "YOUR SERVERIP"
#MysqlAccount = "YOUR USERNAME"
#MysqlPassword = "YOUR PASSWORD"

##### 변경코드
import os 

LogDBName = os.environ["LogDBName"]
MysqlHost = os.environ["MysqlHost"]
MysqlAccount = os.environ["MysqlAccount"]
MysqlPassword = os.environ["MysqlPassword"]
```

이제 도커를 올리면 환경변수값을 가져와서 사용한다.

### Dockerfile을 만들자. 

Dockerfile
```yml
FROM python:2.7-stretch
MAINTAINER "brian.kim" <brian.kim@estsoftinc.com>

RUN apt-get update
RUN pip install mysql-python

RUN mkdir -p /data/LogImporter
COPY LogImporter /data/LogImporter

ENTRYPOINT ["python2","-O","/data/LogImporter/LogImporter22th.py", "--server" ]
```

## 빌드해서 이미지를 만들자. 
cd /docker/log-importer && docker build --tag log-importer .


## docker-compose를 만들자.

```yml
---
version: '3.3'

services:
  logdb:
    image: mysql:5.7
    ports:
      - "3306:3306"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - /data/mysql:/var/lib/mysql
      - ./mysqld.cnf:/etc/mysql/mysql.conf.d/mysqld.cnf:ro

  log-importer-game01:
    image: log-importer
    restart: always
    links:
      - logdb
    depends_on:
      - logdb
    environment:
      LogDBName: DBName
      MysqlHost: HostName
      MysqlAccount: UserName
      MysqlPassword: password
    command: ["game01"]
```

### docker compose로 올리자.

cd /docker/mysql && docker-compose up -d 

### 확인해보자. 아규먼트가 들어갔는지가 중요 
```bash
docker ps -a --no-trunc

CONTAINER ID                                                       IMAGE               COMMAND                                                             CREATED             STATUS              PORTS                    NAMES
6877a09445a022c9b70d95d4fdf7d43e5784301377c800c98c23147656b12116   log-importer        "python2 -O /data/LogImporter/LogImporter22th.py --server game01"   7 minutes ago       Up 7 minutes                                 mysql_log-importer-game01_1

c6552ef42dc9a39d9c8938b38fc0dc46ac18fb50776e0f9a08b0141b23651764   mysql:5.7           "docker-entrypoint.sh mysqld"                                       7 minutes ago       Up 7 minutes        0.0.0.0:3306->3306/tcp   mysql_logdb_1
```
game01이 잘 붙어서 성공이다. 

--no-trunc : 커맨드 내용을 다 보여준다. 

## docker-compose를 추가하여 나머지 서버들도 분석하면된다. 


## Dockerfile에서 cmd와 entrypoint의 차이점 

cmd는 docker run -it --rm zzzz /bin/bash 하면 Dockerfile에 있는 cmd는 무시된다. 

그러나 entryPoint는 아규먼트로 취급된다.  entrypoint에 있는거 다하고 그 후에 "/bin/bash"를 아규먼트로 하나 추가한다. 










