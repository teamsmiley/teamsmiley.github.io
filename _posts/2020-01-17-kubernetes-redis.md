---
layout: post
title: 'kubernetes redis'
author: teamsmiley
date: 2020-01-17
tags: [kubernetes]
image: /files/covers/blog.jpg
category: { kuberentes }
---

redis을 쿠버네티스에 배포 연습
다음과 같은 두가지 사용 패턴이 있다.

1. 단독 서버로 redis를 사용하는경우
1. replicatioh을 사용하는 경우

## 단독 서버를 사용해보자.

docker hub에 있는 기본 redis는 환경변수가 많이 안다. 그리고 설정 파일위치를 못찾겠어서 bitnami/redis 를 사용한다.

환경변수가 많아서 설정하기가 쉽다.

```yml
---
apiVersion: v1
kind: Service
metadata:
  namespace: prod
  name: redis
  labels:
    app: redis
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.79
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: prod
  name: redis
  labels:
    app: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: 'bitnami/redis:latest'
          ports:
            - containerPort: 6379
          env:
            # - name: ALLOW_EMPTY_PASSWORD
            #   value: "yes"
            - name: REDIS_PASSWORD
              value: your-password
```

비밀번호 있이 사용이 가능해졋다. 비번이 필요없으면 ALLOW_EMPTY_PASSWORD를 사용하자.

사용해보자.

```
docker run -it --rm redis redis-cli -h 192.168.0.79 -a your-password
```

```
set mykey myvalue
> OK
get mykey
> "myvalue"
```

## replication

1개의 마스터와 2개의 슬레이브로 구성

vi redis.yml

```yml
---
apiVersion: v1
kind: Service
metadata:
  namespace: prod
  name: redis-master
  labels:
    app: redis-master
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.100
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis-master

---
apiVersion: v1
kind: Service
metadata:
  namespace: prod
  name: redis-slave
  labels:
    app: redis-slave
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.101
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis-slave

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: prod
  name: redis-master
  labels:
    app: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-master
  template:
    metadata:
      labels:
        app: redis-master
    spec:
      containers:
        - name: redis-master
          image: 'bitnami/redis:latest'
          ports:
            - containerPort: 6379
          env:
            - name: REDIS_PASSWORD
              value: your-password
            - name: REDIS_REPLICATION_MODE
              value: master

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: prod
  name: redis-slave
  labels:
    app: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-slave
  template:
    metadata:
      labels:
        app: redis-slave
    spec:
      containers:
        - name: redis-slave
          image: 'bitnami/redis:latest'
          ports:
            - containerPort: 6379
          env:
            - name: REDIS_PASSWORD
              value: your-password
            - name: REDIS_REPLICATION_MODE
              value: slave
            - name: REDIS_MASTER_HOST
              value: redis-master
            - name: REDIS_MASTER_PASSWORD
              value: your-password
```

k apply -f redis.yml

서비스가 2개이다 master ,slave slave는 2개의 파드가 번갈아가면서 대답한다.

테스트해보자.

```bash
docker run -it --rm redis redis-cli -h 192.168.0.100 -a your-password
```

```bash
set mykey myvalue
> OK
get mykey
> "myvalue"
```

slave에서 체크

```bash
docker run -it --rm redis redis-cli -h 192.168.0.101 -a your-password
```

```bash
keys *
> 1) "mykey"
```

잘된다

이제 write는 master에 read는 slave를 쓰면된다.
