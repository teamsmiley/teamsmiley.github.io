---
layout: post
title: 'Docker Mysql Replication' 
author: teamsmiley
date: 2018-08-05
tags: [docker,mysql,replication]
image: /files/covers/blog.jpg
category: {macosx}
---
# docker 로 mysql설치후 리플리케이션 까지. (feat. swarm , docker-machine)

## 목표 
* 2개의 서버에 도커 스웜을 설치하고 컨테이너를 올린다. 
* docker01 : master db 
* docker02 :  slave db를 올린다. 
* 특정 컨테이너는 호스트명으로 구별해서 특정 호스트에서 실행되게 한다. 
* ingress network에서 포트가 중복 안되므로 master는 3306 slave는 3307을 외부에 오픈할 것이다.
* 2개의 mysql이 리플리케이션이 되게 한다. 

## 실서버 설치 

docker01.ur-domain.com : 100.100.100.24
docker02.ur-domain.com : 100.100.100.25

설치후 firewalld는 끄고 yum update는 다 끝내두자. 

## 작업 컴퓨터에서 docker-machine을 사용하여 클러스터를 만든다. 
```
docker-machine -D create \
  --driver generic \
  --generic-ip-address=docker01.domain.com \
  --generic-ssh-key ~/.ssh/id_rsa \
  --generic-ssh-user root \
  docker01

docker-machine -D create \
  --driver generic \
  --generic-ip-address=docker02.domain.com \
  --generic-ssh-key ~/.ssh/id_rsa \
  --generic-ssh-user root \
  docker02
```

-D 옵션은 무슨일이 일어나는지 자세히 보고 싶어서 추가하였다. 

centos에서 root를 사용해서 generic-ssh-user root 도 추가 하였다. 

혹시 에러가 나면 다음을 실행하고 해보자.

```
rm -rf /usr/lib/python2.7/site-packages/backports.ssl_match_hostname-3.*
yum update
```
docker-machine ls 로 설치를 확인하다. 

docker 01 02가 모두 설치 완료 

## 데이터 저장 폴더 만들기 

각 서버에 mysql데이터 저장공간인 /data/mysql 을 생성하도록 하자. (컨테이너 안에 데이터를 저장하게 하면 안됨. 도커재시작시 데이터 사라짐)
```
docker-machine ssh docker01 mkdir -p /data/mysql
docker-machine ssh docker02 mkdir -p /data/mysql
```

## 도커 스웜 클러스터를 생성하자. 

### docker01 swarm init
```bash
eval $(docker-machine env docker01)
docker swarm init 
# docker swarm join --token SWMTKN-1-3yq9e4n939qp2rlvw1p02sqxscxgnv1v3jf2eqxqfp53j1r9t2-0pz62kiaj2700pldxikwzn0cq 192.168.1.24:2377
```

### docker02 swarm node 
```
eval $(docker-machine env docker02)
docker swarm join --token SWMTKN-1-3yq9e4n939qp2rlvw1p02sqxscxgnv1v3jf2eqxqfp53j1r9t2-0pz62kiaj2700pldxikwzn0cq 192.168.1.24:2377
```
### docker01 에서 노드를 확인하자.
```
eval $(docker-machine env docker01)
docker node ls
```
2대가 보이고 한대는 매니저 한대는 노드로 보인다. 


## docker-compose셋팅 (macosx)
현재 작업하는 컴퓨터에서 docker-compose파일을 만들어야한다. 
```
cd ~/docker01/mysql
vi docker-compose.yml
```

```yml
---
version: "3.3"

services:
  master:
    image: mysql:5.5
    ports:
      - "3306:3306"
    volumes:
      - /data/mysql:/var/lib/mysql
      - /docker/mysql/ftpuser/conf.d/master.cnf:/etc/mysql/conf.d/master.cnf
    environment:
      MYSQL_ROOT_PASSWORD: XXXXXXXX
      MYSQL_DATABASE: ftp1
      MYSQL_USER: urid
      MYSQL_PASSWORD: XXXXXXXX
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.hostname == docker01

  slave:
    image: mysql:5.5
    ports:
      - "3307:3306"
    volumes:
      - /data/mysql:/var/lib/mysql
      - /docker/mysql/ftpuser/conf.d/slave.cnf:/etc/mysql/conf.d/slave.cnf
    environment:
      MYSQL_ROOT_PASSWORD: XXXXXXXX
      MYSQL_DATABASE: ftp1
      MYSQL_USER: urid
      MYSQL_PASSWORD: XXXXXXXX
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.hostname == docker02

```
* master db mysql 설정
```
mkdir conf.d
vi conf.d/master.cnf
[mysqld]
log-bin=mysql-bin  
server-id=1  
```
* slave db mysql 설정

```
vi conf.d/slave.cnf
[mysqld]
server-id=2  
```

## 설정파일을 각 노드에 복사하자
```
docker-machine ssh docker01 mkdir -p /docker/mysql/ftpuser
docker-machine ssh docker02 mkdir -p /docker/mysql/ftpuser
docker-machine scp -r . docker01:/docker/mysql/ftpuser
docker-machine scp -r . docker02:/docker/mysql/ftpuser
```

## docker stack 으로 배포하기
```bash
eval $(docker-machine env docker01)
docker stack deploy -c ftpuser/docker-compose.yml ftpuser

docker service ls 
docker service logs ftpuser_master
```
배포시 에러 나면 아래처럼 확인해보자.
```
docker service ps --no-trunc ftpuser_master
docker service ps --no-trunc ftpuser_slave
```

## replication
master에서 백업을 해서 slave에 복구해주고 replication설정 해주면 된다.

### master 디비
백업하고 
```bash
docker ps -a  # id 확인
docker exec fe84c9117641 mysqldump -u root -pXXXXXXXX ftp1 > backup.sql
```
mysql에 접속해서 필요한 정보를 확인한다. 
```
docker exec -it  fe84c9117641 mysql -u root -pXXXXXXXX 
SHOW MASTER STATUS; 
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000026 |      107 |              |                  |
+------------------+----------+--------------+------------------+ 
```

이와같은결과가 나오면 적어둔다. 복구시 사용한다.

slave로 복사 
```
scp -r backup.sql root@docker02:/data
```

### slave 에 복구

backup.sql을 가져와서 mysql을 복구한다. 
```
cat /data/backup.sql | docker exec -i 8b27cd107ede mysql -u root -pXXXXXXXX ftp1
```

mysql에 접속후 다음 스크립트를 실행한다. 

```
docker exec -it 8b27cd107ede mysql -u root -pXXXXXXXX

CHANGE MASTER TO
master_host = 'master',
master_user = 'root',
master_password = 'XXXXXXXX',
master_log_file = 'mysql-bin.000026',
master_log_pos = 107;

START SLAVE;
```
bin 과 pos는 위에서 찾야뒀던 값을 넣어준다. 

### 슬래이브가 잘 되는지 확인하자. 

```sql
show slave status \G
```

## 동작 확인 

마지막으로 master 에 값을 하나 더 넣고 slave에 잘 들어가는지 확인하면된다. 
```
insert into accounts (username,pass) values ('aaaaaaaaaaaa','aaaaaaaaaaaaaaaa');
select * from accounts where username='aaaaaaaaaaaa';
delete from accounts where username='aaaaaaaaaaaa';
```

## 도커 스웜 컨테이너 구성을 웹으로 보자. 
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer


## 추가로 해야할일 

* 기존에 있는 도커면 어덯게 도커머신에 등록하는가?
* docker-machine을 하면 실서버 도 지워져 버린다.


