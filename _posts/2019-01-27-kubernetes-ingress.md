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

하나의 아이피에 각각의 서비스로 보내주고 싶다.

아래의 서비스가 있다.

## 기본 설치 
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

## nodeport 사용

vi ingress-nodeport.yml

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: ingress-nodeport
  labels:
    service-name: ingress-nodeport
spec:
  containers:
  - name: ingress-nodeport
    image: asbubam/hello-node
    readinessProbe:
      httpGet:
        path: /
        port: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080

---
apiVersion: v1
kind: Service # service생성
metadata:
  name: ingress-nodeport
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 31000
  selector:
    service-name: ingress-nodeport

---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nodeport
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
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
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

kubectl create -f ingress-nodeport.yml



잘됬는지 체크 
```bash
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx 
kubectl get svc   #open된 포트 확인하자 31000
```

인그레스 설정을 추가하자.

vi ingress-nodeport-config.yml
```yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: np.publishapi.com
    http:
      paths:
      - path: /
        backend:
          serviceName: ingress-nodeport
          servicePort: 8080
```
kubectl create -f kubectl create -f ingress-nodeport-config.yml

확인해보자.

curl http://192.168.0.192:31000 

vi /etc/hosts
```
192.168.0.192 np.publishapi.com
```
curl http://np.publishapi.com:31000 (ok)

도메인 이름으로 서비스가 되는것을 알수 있다.

## 로드발란스 타입의 ingress 서비스로 바꿔보자.

metallb를 꼭 설치하시기바랍니다.

```
vi ingress-loadbalance.yml
```
```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: ingress-loadbalance
  labels:
    service-name: ingress-loadbalance
spec:
  containers:
  - name: ingress-loadbalance
    image: asbubam/hello-node
    readinessProbe:
      httpGet:
        path: /
        port: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080

---
apiVersion: v1
kind: Service # service생성
metadata:
  name: ingress-loadbalance
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: ingress-loadbalance

---
apiVersion: v1
kind: Service
metadata:
  name: ingress-loadbalance
  namespace: ingress-nginx # 이부분 수정말자
  labels:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정말자
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정말자
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.84
  ports:
    - name: http
      port: 8080
      targetPort: 8080
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정말자
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정말자
```

적용
```
kubectl create -f ingress-loadbalance.yml
```

```
vi ingress-loadbalance-config.yml
```
```yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: lb.publishapi.com
    http:
      paths:
      - path: /
        backend:
          serviceName: ingress-loadbalance
          servicePort: 8080
```

적용
```
kubectl create -f ingress-loadbalance-config.yml
kubectl get svc -n ingress-nginx 
```

vi /etc/hosts
```
192.168.0.84 lb.publishapi.com
```

curl http://192.168.0.84  not working

curl http://lb.publishapi.com 

로드발란스로 동작한다.

* 중요 

헷갈린 부분이 있어서 정리해둔다. 

일단 서비스로 올리는 컨테이너는 # 이부분 아래 설명 참고 - A 이부분을 바꾸지 말기 바란다. 

바꿀수도 있지만 일이 복잡해진다.  이름만 바꾸고 쓰자. 

그러나 실제 연결 되는 서비스는 namespace가 위와 달라도  서비스명으로 그냥 연결할수 있다. 아래 스피네커 붙이는 부분 참고 




로드발란스 타입으로 만듭니다. metallb가 꼭 필요합니다. 앞쪽문서를 참고하세요

```bash
kubectl create namespace ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
# using nodeport
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml

vi ingress-nodeport.yml
```

```yml
# pod 생성
---
apiVersion: v1
kind: Pod
metadata:
  name: hello-ingress
  labels:
    service-name: hello-ingress
spec:
  containers:
  - name: hello-ingress
    image: asbubam/hello-node
    readinessProbe:
      httpGet:
        path: /
        port: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080

# service생성
---
apiVersion: v1
kind: Service 
metadata:
  name: hello-ingress
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: hello-ingress

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: publishapi.com
    http:
      paths:
      - backend:
          serviceName: hello-ingress
          servicePort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: hello-ingress
  namespace: ingress-nginx # 이부분 수정없이 사용하자
  labels:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정없이 사용하자
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정없이 사용하자
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.84
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx # 이부분 수정없이 사용하자
    app.kubernetes.io/part-of: ingress-nginx # 이부분 수정없이 사용하자
```

적용하고 확인해보자.
```
kubectl create -f ingress-nodeport.yml
kubectl get svc -n ingress-nginx 
```

vi /etc/hosts
```
192.168.0.83 publishapi.com
```

curl http://192.168.0.83  not working

curl http://publishapi.com 

로드발란스로 동작한다.

* 중요 

헷갈린 부분이 있어서 정리해둔다. 

일단 서비스로 올리는 컨테이너는 # 이부분 아래 설명 참고 - A 이부분을 바꾸지 말기 바란다. 

바꿀수도 있지만 일이 복잡해진다.  이름만 바꾸고 쓰자. 

그러나 실제 연결 되는 서비스는 namespace가 위와 달라도  서비스명으로 그냥 연결할수 있다. 아래 스피네커 붙이는 부분 참고 










