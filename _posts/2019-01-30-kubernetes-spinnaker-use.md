---
layout: post
title: 'spinnaker 사용' 
author: teamsmiley
date: 2019-01-30
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes,spinnaker}
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
  namespace: prod
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
  namespace: prod
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
        - image: 'registry.publishapi.com:5000/auth-server:${trigger["tag"]}' # 이거 중요 트리거에서 넘겨준 정보를 가지고 빌드한다.
          name: auth
      imagePullSecrets:   # 이부분 추가
        - name: my-registry # 이부분 추가
```

${trigger["tag"]} 이 부분이 트리거에서 넘겨주는 값을 가지고 빌드를 하는 부분

kubernetes secret를 만든다. namespace를 잊지말것

```
kubectl create secret docker-registry my-registry \
--docker-server=https://registry.publishapi.com:5000 \
--docker-username=<your-id> \
--docker-password=<your-pass> \
--docker-email=brian@publishapi.com \
--namespace prod
```

이제 파이프라인을 빌드하면 서비스가 생성되고 기존 replicaset,pod들은 전부 삭제하자.

### add stage for disable(manifest)

add stage >> disable(manifest) 

![]({{site_baseurl}}/assets/spinnaker-11.png)
![]({{site_baseurl}}/assets/spinnaker-12.png)

여기에서 클러스터 부분이 드롭다운이 안나타나면  일단 disable 없이 파이프라인을 실행한 후 로드발란스에 replica set이 생긴 후 다시 이부분을 진행해보면 리스트가 나오는것을 볼수 있을것이다.

### 실행 
docker image를 새로운 태그로 푸시하면 스피네커가 인식하여  트리거가 동작하고 배포를 시작하면 완료된 것이다.


