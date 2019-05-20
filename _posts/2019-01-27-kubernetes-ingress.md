---
layout: post
title: 'kubernetes ingress nginx' 
author: teamsmiley
date: 2019-01-27
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# ingress-nginx

하나의 아이피에 각각의 서비스로 보내주고 싶다. 구성은 다음 그림과 같다. 

![]({{site_baseurl}}/assets/ingress-0.png)

## 기본 pod and service 생성

vi hello-node.yml

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: hello-node
  labels:
    service-name: hello-node
spec:
  containers:
  - name: hello-node
    image: asbubam/hello-node
---
apiVersion: v1
kind: Service 
metadata:
  name: hello-node
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: hello-node
```

```
kubectl create -f hello-node.yml
```

잘됬는지 체크 
```bash
kubectl get pods 
kubectl get svc
```

현재까지 구성은 다음과 같다. 

![]({{site_baseurl}}/assets/ingress-1.png)


## ingress 설치 
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

## 로드발란스 타입의 ingress 서비스

metallb를 꼭 설치하시기바랍니다.

* ingress-nginx 서비스는 namespace가 꼭 ingress-nginx 로 해야한다.
* ingress-config에서는 연결하는 서비스에 namespace와 같은 곳에 ingress를 설치 해야한다. 

```
vi ingress-service.yml
```
```yml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-test
  namespace: ingress-nginx # 이부분 수정 하지 말자.
  labels:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정 하지 말자.
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정 하지 말자.
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.84
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정 하지 말자.
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정 하지 말자.
```
```
kubectl create -f ingress-service.yml
```

```
vi dev-ingress.yml
```
```yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  namespace: default # 연결되는 서비스의 네임스페이스와 같아야한다.
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: lb.publishapi.com
    http:
      paths:
      - backend:
          serviceName: hello-node
          servicePort: 8080
```
```
kubectl create -f ingress-domain-config.yml
```

```
vi /etc/hosts
```
```
192.168.0.84 lb.publishapi.com
```

curl http://192.168.0.84  

```html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.15.8</center>
</body>
</html>
```

curl http://lb.publishapi.com 
```
Hello World!
```

현재까지 구성은 다음과 같다.

![]({{site_baseurl}}/assets/ingress-3.png)

나머지는 ingress-service.yml 에서 Ingress Rule 만 계속 추가하면 된다.

