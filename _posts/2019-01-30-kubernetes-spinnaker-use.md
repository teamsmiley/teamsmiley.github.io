---
layout: post
title: 'kubernetes spinnaker 사용' 
author: teamsmiley
date: 2019-01-30
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# Spinnaker 사용

## blue green 배포 적용 

<https://www.spinnaker.io/guides/user/kubernetes-v2/traffic-management/#sample-bluegreen-pipeline>

여기를 잘 참고하면 된다. 

영어가 불편하신분을 위해 설명을 하면  

1. application을 만들자.  
2. load balancer를 만들자. (service)
3. pipeline을 추가한다. 

아직 ingress가 스피네커를 통해서 생성이 안되는듯 보여 수동으로 생성했다. (방법 아시는분은 댓글좀..)

해보자.

## spinnaker ui에 접속하자. 

앞 문서를 참고하여 http://spin.publishapi.com으로 접속

## Add Application 

![]({{site_baseurl}}/assets/spinnaker-01.png)

### Add Service (load balancer)

![]({{site_baseurl}}/assets/spinnaker-02.png)

![]({{site_baseurl}}/assets/spinnaker-03.png)

```yml
kind: Service
  apiVersion: v1
  metadata:
    name: auth
    namespace: auth-live
  spec:
    selector:
      app: auth
    ports:
    - protocol: TCP
      port: 80
```

## 파이프 라인을 추가하자. 

![]({{site_baseurl}}/assets/spinnaker-04.png)

![]({{site_baseurl}}/assets/spinnaker-05.png)

### add triger 
![]({{site_baseurl}}/assets/spinnaker-06.png)

configurtion ==> 트리거 적용 >> docker registry  >> name >> image >> enable trigger

태그가 번호가 꼭 바뀌어야 이 트리거가 실행된다.

![]({{site_baseurl}}/assets/spinnaker-07.png)

### add stage for deploy

![]({{site_baseurl}}/assets/spinnaker-08.png)

![]({{site_baseurl}}/assets/spinnaker-09.png)

add stage >> Deploy(Manifest) >> text 

![]({{site_baseurl}}/assets/spinnaker-10.png)

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    strategy.spinnaker.io/max-version-history: '2' # 최대 보관 이미지 갯수 롤백이 최근 1번째까지 가능하게 된다.
    traffic.spinnaker.io/load-balancers: '["service auth"]' #위에 만들어둔 서비스 붙인다. 이게 없으면 disable이 안된다. 
  labels:
    tier: auth
  name: auth
  namespace: auth-live
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: auth
  template:
    metadata:
      labels:
        tier: auth
    spec:
      containers:
        - image: 'registry.xgridcolo.com:5000/auth-server:${trigger["tag"]}' # 이거 중요 트리거에서 넘겨준 정보를 가지고 빌드한다.
          name: auth
```

${trigger["tag"]} 이 부분이 트리거에서 넘겨주는 값을 가지고 빌드를 하는 부분

파이프라인을 빌드하면 서비스가 생성되고 기존 서비스는 disable된다.

2개 이상의 pod들은 전부 삭제된다.

### add stage for disable(manifest)

add stage >> disable(manifest) 

![]({{site_baseurl}}/assets/spinnaker-11.png)
![]({{site_baseurl}}/assets/spinnaker-12.png)

### manual run 

이제 실행해본다.

## Error
* ImagePullError : 노드가 도커 레지스트리에 접근을 못해서 생김 
```bash
docker login registry.publishapi.com:5000 
docker pull registry.publishapi.com:5000/auth:100 #100번째 이미지 다운되는지 확인
```
