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

로드발란스 타입으로 만듭니다. metallb가 꼭 필요합니다. 앞쪽문서를 참고하세요

```bash
kubectl create namespace ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
# using nodeport
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml

vi hello-ingress.yml
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
kubectl create -f hello-ingress.yml
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










