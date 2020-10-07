---
layout: post
title: "kubernetes helm"
author: teamsmiley
date: 2020-10-06
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

# helm

helm3부터는 tilder를 설치안해도 되서 좋다.

## 설치

macos에서 설치하면

```bash
brew install helm
helm version
```

## 패키지 찾기

```bash
helm search hub # 여러 저장소들에 있는 헬름 차트들을 포괄하는 헬름 허브를 검색한다.
helm search hub cert-manager
helm search repo # helm repo add를 사용하여 로컬 헬름 클라이언트에 추가된 저장소들을 검색한다. 검색은 로컬 데이터 상에서 이루어지며, 퍼블릭 네트워크 접속이 필요하지 않다.
helm search repo cert-manager
```

## repo

```bash
helm repo add NAME URL
helm repo add jetstack https://charts.jetstack.io
helm repo list
helm repo remove NAME
helm repo update
```

## 패키지 설치

패키지 설치 명령어도 2와 조금 달라져서 --name 은 동작하지 않는다.

```bash
# helm install NAME PACKAGE

helm install my-cert-manager jetstack/cert-manager
# --generate-name : 랜덤한 숫자들을 자동으로 붙여준다.
helm install jetstack/cert-manager --generate-name --set installCRDs=true
```

## 패키지 리스트

```bash
helm list --all # 여기에 이름이 나온다. 그 이름으로 제거한다.
```

## 패키지 삭제

```bash
# helm uninstall NAME
helm uninstall cert-manager-1601943552 # 위 리스트에서 나온 이름
```
