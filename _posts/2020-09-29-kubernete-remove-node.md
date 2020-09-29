---
layout: post
title: "kubernete remove node"
author: teamsmiley
date: 2020-09-29
tags: [cicd]
image: /files/covers/blog.jpg
category: { kube }
---

# kubernetes에서 필요 없는 노드 삭제

## 더이상 pod가 할당 되지 않게

```bash
kubectl cordon master03
```

포드가 노드에 정상적으로 스케쥴링될 수 있게 하기 위해서는 uncordon

## 기존 포드들을 다른곳으로 이동

```bash
kubectl drain master03

node/master03 cordoned
error: unable to drain node "master03", aborting command...

There are pending nodes to be drained:
 master03
error: cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/kube-proxy-v7jz8, kube-system/weave-net-f52sc, metallb-system/speaker-gzq42
```

에러가 난다. daemonset을 못지운다는

에러에 나온 옵션 추가

```bash
kubectl drain master03 --ignore-daemonsets
```

잘 된다.

## 노드리스트에서 제거

```bash
kubectl delete node master03
kubectl get list
```

## 노드도 초기화

```bash
ssh master03
kubeadm reset
```

완료
