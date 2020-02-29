---
layout: post
title: 'kubernetes upgrade' 
author: teamsmiley
date: 2020-02-29
tags: [kubernetes]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes version upgrade

마스터 노드를 먼저 하고 워커 노드를 나중에 한다.

kubeadm 을 먼저 업그레이드하고 kubelet kubectl을 나중에 업그레이드한다.

## kubeadm

### master01
```bash
ssh master01

yum list --showduplicates kubeadm --disableexcludes=kubernetes

yum install -y kubeadm-1.17.3-0 --disableexcludes=kubernetes

kubeadm version #확인

kubectl drain master01 --ignore-daemonsets # master01을 클러스터에서 뺀다. 업그레이드후 나중에 다시 붙이자.

kubeadm upgrade plan # 업그레이드 플랜내용을 보여준다. 마지막 커맨들를 실행하자.

kubeadm upgrade apply v1.17.3
```

한참 뭐를 한다. 

마지막으로 
```bash
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.17.0". Enjoy!
```

이것이 나오면 맞는것이다.

```bash
kubectl uncordon master01 # 다시 클러스터에 붙인다.
```

### master02 master03

kubeadm 설치후
```bash
yum install -y kubeadm-1.17.3-0 --disableexcludes=kubernetes
kubeadm version #확인
```

`kubeadm upgrade apply` 대신 `kubeadm upgrade node` 이걸 한다.

## kubelet

### master01 02 03 

```bash
yum install -y kubelet-1.17.x-0 kubectl-1.17.3-0 --disableexcludes=kubernetes
systemctl restart kubelet
```

## Worker node 

기본적으로는 같다. 일단 클러스터에서 빼고 업그레이드후 다시 넣는다.

```bash
kubectl drain node13 --ignore-daemonsets # 클러스터에서 제거 
yum install -y kubeadm-1.17.3-0 --disableexcludes=kubernetes # 설치
kubeadm upgrade node # 설정 업그레이드

yum install -y kubelet-1.17.3-0 kubectl-1.17.3-0 --disableexcludes=kubernetes # 설치
systemctl restart kubelet # 재시작

kubectl uncordon node13
```

## 확인하자.

```bash
k get nodes

NAME       STATUS   ROLES    AGE    VERSION
master01   Ready    master   144d   v1.17.3
master02   Ready    master   144d   v1.17.3
master03   Ready    master   144d   v1.17.3
node11     Ready    <none>   144d   v1.17.3
node12     Ready    <none>   144d   v1.17.3
node13     Ready    <none>   42m    v1.17.3
node14     Ready    <none>   144d   v1.17.3
```

나머지 노드들도 하나씩 해주면 된다.

## 배운점 
* kube cluster에서 yum update를 하지말자.. 커널도 업데이트되고 막 뭐가 설치되면 나중에 알수가 없다. 패키지가 200개가 깔리기도 함.
* 전체노드를 동시에 업데이트하지 말자 하나씩 클러스터에서 빼서 하고 다시 넣고 해야한다.
