---
layout: post
title: 'kubernetes add node' 
author: teamsmiley
date: 2020-02-29
tags: [kubernetes]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes node 추가 

## kube cluster nodelist 에서 삭제 
```bash
kubectl get nodes

NODENAME=node13
kubectl drain $NODENAME
# kubectl drain $NODENAME --ignore-daemonsets --delete-local-data
kubectl delete node $NODENAME
kubectl get nodes
```
이제 클러스터에서 빠졌다.

## 노드 재설치  

###  새로 노드를 재설치 
kubernetes를 맞는 버전을 설치해야한다. 그냥 설치하면 최신 버전이 자동설치 되버린다.

```bash
yum remove kubeadm kubelet kubectl -y
yum install kubeadm-1.16.1 kubelet-1.16.1 kubectl-1.16.1 -y
```

### 기존 노드를 사용하려면
```bash
kubeadm reset #delete kube info on this node
```
이러면 된다.

## 재설치 한 노드를 다시 클러스터에 join하려면 
```bash
kubeadm token list # token이 있으면 사용한다.
kubeadm token create # 만약 없으면 만들어준다.
> 1svjiu.7e0xw9v03o9xw

# discovery-token-ca-cert-hash 확인
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
> ba6c2986c3dd2e527554ee056b63b4a1826bd9b4a6449dd3ccbc50

# 노드 join하자.
ssh node13
kubeadm join 192.168.0.100:6443 --token 1svjiu.7e0xw9v03o9 --discovery-token-ca-cert-hash sha256:ba6c2986c3dd2e527554ee056b63b4a1826bd9b4a6449dd3ccbc50
```

## 추가된건지 확인해보자.
```bash
kubectl get nodes
```



