---
layout: post
title: 'Docker Mysql Replication' 
author: teamsmiley
date: 2018-08-05
tags: [docker,mysql,replication]
image: /files/covers/blog.jpg
category: {macosx}
---
# docker 로 mysql설치후 리플리케이션 까지.

현재 호스트명이 master와 slave 서버가 있다. 각각의 아이피는 192.168.1.24 192.168.1.29 이다. 

mysql의 데이터파일을은 각각의 서버에 /data/mysql에 저장하게 하자. (컨테이너 안에 저장하게 하면 안됨 도커재시작시 데이터 사라짐)

도커를 스웜을 사용해서 mysql을 올리면 어느 서버에 올라갈지 모른다. 그래서 특정 이미지는 특정 서버에 동작하게 한다. 

컨테이너끼리 연결을 해서 리플리케이션을 해야하기 때문에 overlay network를 사용해야해서 swarm을 설치한다. 

ingress network를 사용하기 때문에 하나의 docker-compose파일에서 호스트의 같은 포트를 오픈할수가 없다.  그러므로 우리는 master는 3306 slave는 3307을 외부에 오픈할 것이다.

## swarm 설치 
스웜을 설치하면 여러개의 노드가 하나의 아이피로 대표 될수 있다

### master

docker swarm init --advertise-addr 192.168.99.24

결과로 나온 스크립트를 복사해둔다. 
```
docker swarm join --token SWMTKN-1-26qpppygfw3jc4yyy0ln5rtibqkrrsdbtkae656r15281md8gr-8tgq0y6hrf554t02suiu4ut4j 192.168.1.24:2377
```

mkdir -p /data/mysql 

데이터 폴더를 만든다. 

### slave

위에 복사해둔 스크립트를 실행하면 스웜 클러스터에 조인이 된다. 

mkdir -p /data/mysql

## docker-compose셋팅 

### master에서 
```
mkdir -p /docker/mysql/conf.d
vi /docker/mysql/docker-compose.yml
```

```yml
---
version: '3.3'

services:
  master:
    image: mysql:5.5
    ports:
      - "3306:3306"
    restart: always
    volumes:
      - /data/mysql:/var/lib/mysql
      - ./conf.d/master.cnf:/etc/mysql/conf.d/master.cnf
    environment:
      MYSQL_ROOT_PASSWORD: urpass
      MYSQL_DATABASE: ftp1
      MYSQL_USER: testuser
      MYSQL_PASSWORD: urpass
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
        - node.hostname == master

  slave:
    image: mysql:5.5
    ports:
      - "3307:3306"
    restart: always
    volumes:
      - /data/mysql:/var/lib/mysql
      - ./conf.d/slave.cnf:/etc/mysql/conf.d/slave.cnf
    environment:
      MYSQL_ROOT_PASSWORD: urpass
      MYSQL_DATABASE: ftp1
      MYSQL_USER: testuser
      MYSQL_PASSWORD: urpass
    links:
      - master
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
        - node.hostname == slave
```

vi /docker/mysql/conf.d/master.cnf
```
[mysqld]
log-bin=mysql-bin  
server-id=1  
```
vi /docker/mysql/conf.d/slave.cnf
```
[mysqld]
server-id=2  
```

* constraints로 특정 서버를 호스트이름으로 정해줘서 master 서버에서는 master 컨테이너가 뜨게 처리했다. 
마찬가지로 slave 컨테이너는 슬레이브 호스트에서만 실행되게 햇다. 

* slave에서는 link를 이용해서 master를 링크하게 해두었다. replication할때 호스트명이 사용 되기 때문이다.

## docker stack으로 배포하기

```
docker stack deploy -c docker-compose.yml mysql
docker service ls 
docker service logs mysql_master
```

## swarm visual monitor 설치하기 

```
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```

## replication
master에서 백업을 해서 slave에 복구해주고 replication설정 해주면 된다.
### master 

mysql에 접속해서 디비를 만든다.
```
docker exec -it mysql_master mysql -u root -p 
create databases ftp1;
exit;
```

백업을 해서 슬래이브에 복구해줄것임 
```
docker exec mysql_master mysqldump -u root -pxxxxxx ftp1 > backup.sql
```

mysql에 접속해서 필요한 정보를 확인한다. 
```
docker exec -it mysql_master mysql -u root -p 
SHOW MASTER STATUS; 
```
| mysql-bin.000013 |      107 |      

이와같은결과가 나오면 적어둔다. 복구시 사용한다.

### slave

backup.sql을 가져와서 mysql을 복구한다. 
```
cat /docker/mysql/backup.sql | docker exec mysql_slave mysql -u root -pXXXXX ftp1
```

mysql에 접속후 다음 스크립트를 실행한다. 

```
docker exec -it mysql_slave mysql -u root -p
```

```sql
CHANGE MASTER TO
master_host = '192.168.1.24',
master_user = 'root',
master_password = 'urpass',
master_log_file = 'mysql-bin.000013',
master_log_pos = 107;

START SLAVE;
```

bin 과 pos는 위에서 찾야뒀던 값을 넣어준다. 

### 슬래이브가 잘 되는지 확인하자. 

```sql
show slave status \G
```

에러 없으면 잘 된다

## 동작 확인 

마지막으로 master 에 값을 하나 더 넣고 slave에 잘 들어가는지 확인하면된다. 






