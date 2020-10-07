---
layout: post
title: "kubernetes - prometheus & grafana"
author: teamsmiley
date: 2020-10-08
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>
1. <https://teamsmiley.github.io/2020/10/05/kubernetes-cert-manager/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-metallb-ingress-nginx/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-helm/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-prometheus-grafana/>

# kubernetes - prometheus & grafana

kubernetes 모니터링을 해보자.

참고 : <https://github.com/coreos/kube-prometheus#quickstart>

## install

```bash
cd ~/Desktop
git clone https://github.com/coreos/kube-prometheus.git
cd ~/Desktop/kube-prometheus

# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup
kubectl create -f manifests/

# 확인
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done

## 삭제
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
kubectl delete --ignore-not-found=true -f manifests/ -f manifests
```

monitoring namespace에 설치가 다 되었다.

## prometheus dashboard 확인

포트 포워딩을 한다.

```bash
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090
```

웹사이트로 확인한다.

<http://localhost:9090>

![]({{ site_baseurl }}/assets/2020-10-07-05-25-09.png)

## grafana 대시보드

포트 포워딩을 한다.

```bash
kubectl --namespace monitoring port-forward svc/grafana 3000
```

웹사이트로 확인한다.

<http://localhost:3000>

![]({{ site_baseurl }}/assets/2020-10-07-05-25-09.png)

## 서비스를 만들어서 외부에 오픈하자.

### prometheus

```bash
vi ~/Desktop/prometheus-external-service
```

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    prometheus: k8s
  name: prometheus-k8s-ext
  namespace: monitoring
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.2.98
  ports:
    - name: web
      port: 80
      targetPort: web
  selector:
    app: prometheus
    prometheus: k8s
  sessionAffinity: ClientIP
```

k apply -f ~/Desktop/prometheus-external-service

### grafana

vi ~/Desktop/grafana-external-service.yaml

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: grafana-ext
  name: grafana-ext
  namespace: monitoring
spec:
  type: LoadBalancer # 여기 추가
  loadBalancerIP: 192.168.2.97 # 여기 추가
  ports:
    - name: http
      port: 80
      targetPort: http
  selector:
    app: grafana
```

```bash
k apply -f ~/grafana-external-service.yaml
```

<http://192.168.2.97/login> 확인 성공

일단 admin/admin으로 로그인을 하면 비번을 변경하라고한다. 원하는 비번으로 변경한다.

manage >> dash board >> default 에 보면 리스트가 많이 나온다 그것중 마음에 드는거으로 클릭해보면 다음같은 화면이 나온다.

![]({{ site_baseurl }}/assets/2020-10-07-06-20-40.png)

잘 구성해보면 된다.
