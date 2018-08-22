---
layout: post
title: 'Docker Tip 02' 
author: teamsmiley
date: 2018-08-17
tags: [docker,docker-machine]
image: /files/covers/blog.jpg
category: {macosx}
---

# Docker Tip 2 

기본적인 도커 사용법을 배우고 난후 뭐를 해볼까 싶어서 진행하다 생긴 이슈를 적어본다.

## 스웜을 설치해서 사용할때 특정 노드에 특정 서비스를 실행하고 싶다. 
docker-compose에서 constraints를 사용해서 처리한다. 
```yml
---
version: "3.3"

services:
  docker01:
    image: mysql:5.5
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.hostname == docker01
```

이러면 docker01이라는 호스트에서만 배포가 된다. 

## 그럼 특정 노드가 아니라 특정 타입 예를 들면 32기가 장비에서만 서비스를 실행해라고는 어떻게 할까?
```bash
docker node update --label-add <key>=<value> <node-id>
docker node update --label-add memory=32 docker01
```
docker node update명령어를 가지고 레이블을 추가할수 있습니다. 

잘 추가 됬는지 확인해보자 

```bash
docker node inspect docker01
```

```json
[
    {
        "ID": "z6dghvte0frwanvpc40aagh20",
        "Version": {
            "Index": 3856
        },
        "CreatedAt": "2018-08-12T19:31:59.158338895Z",
        "UpdatedAt": "2018-08-18T17:23:15.142475742Z",
        "Spec": {
            "Labels": {
                "memory": "32"
            },
            "Role": "manager",
            "Availability": "active"
        },
        "Description": {
            "Hostname": "docker01",
            "Platform": {
                "Architecture": "x86_64",
                "OS": "linux"
            },
            "Resources": {
                "NanoCPUs": 16000000000,
                "MemoryBytes": 33554534400
            },
            ...
        },
        "Status": {
            "State": "ready",
            "Addr": "192.168.1.24"
        },
        "ManagerStatus": {
            "Leader": true,
            "Reachability": "reachable",
            "Addr": "192.168.1.24:2377"
        }
    }
]
```

Spec에 보면 Label이 잘 들어가 있다.

이제 이 spec을 기준으로 docker-compose.yml에서 사용할수 있다. 
```yml
version: '3.3'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - node.labels.memory == 32
          - node.labels.disk == ssd
          - engine.labels.operatingsystem == ubuntu 14.04
```

## 라벨은 노드에만 붙일수 있는 것인가?

스웜모드에서 노드에 붙일수 있고 Docker Engine에도 붙일수가 있다.

engine.labels.operatingsystem 등으로도 배포할곳을 지정할수 있다. 

정확한걸 알고 싶으면 docker inspector를 해보면 알 수 있다. 

참고 <https://docs.docker.com/ee/ucp/admin/configure/add-labels-to-cluster-nodes/#deploy-a-service-with-constraints>

## 여러대의 도커 장비를 관리하려면 어떻게 해야하나?

docker-machine 을 사용하자. 

centos 7 만 설치된 서버가 있고 ( yum update는 꼭 해둘것) 랩탑에서 다음처럼 해보자 

```
docker-machine -D create \
  --driver generic \
  --generic-ip-address=docker01.xxx.com \
  --generic-ssh-key ~/.ssh/id_rsa \
  --generic-ssh-user root \
  docker01
```

서버에 접속해서 도커를 설치하고 연결을 한다. 
yum업데이트는 위 커맨드 전에 해두면 좋다. 아닌경우에는 yum updata때문에 프로세스가 죽은듯 보일수도 있다. 

이제 docker-machine ls 로 보면 node가 보인다. 


## 관련 명령어 정리
```bash
docker-machine ls
docker-machine env docker01 ==> docker01환경 변수 보여줌.
eval $(docker-machine env docker01) ==> docker 01에 로컬 커맨드를 연결 
docker-machine ls ==> active가 뭐가 되잇는지 *로 표시된다.
docker-machine active  ==> active를 보여준다.
docker-machine ssh docker01 ==> ssh로 서버에 접속한다. 
docker-machine regenerate-certs docker01 ==> 혹시 cert에러시 재생성 
docker-machine env -u ==> 연결된 도커와 연결을 끊고 로컬로 가려면
docker-machine inspect docker01
docker-machine provision docker01  ==> 초기화
docker-machine upgrade docker01 ==> 업데이트
docker-machine create
docker-machine stop 
docker-machine start 
docker-machine restart
docker-machine kill
docker-machine status
docker-machine rm 
docker-machine ls --filter label=cluster=dev ==> 필터링 해서 사용할때
docker-machine ip docker01 ==> 도커 ip를 알고 싶을때
docker-machine  create --driver virtualbox pxe  ==> local에 virtualbox로 docker를 생성할때
```

