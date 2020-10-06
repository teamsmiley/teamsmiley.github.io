---
layout: post
title: "kubespray - 04 Kubernetes Multi Cluster"
author: teamsmiley
date: 2020-10-04
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# kubespray - 04 multi cluster 관리

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>
1. <https://teamsmiley.github.io/2020/10/05/kubernetes-cert-manager/>

이제 kube spray로 2개 이상의 클러스터를 생성시 관리방법을 이야기해보자.

## download cluster 1 config

랩탑에 scp로 1번째 클러스터에 설정 파일을 가져온다.

```
mkdir ~/.kube
scp c1-master:/etc/kubernetes/admin.conf ~/.kube/c1-config
```

필요한 부분을 수정하자.

```yml
apiVersion: v1
clusters:
  - cluster:
      certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0F
      server: https://192.168.0.100:6443
    name: kube-c1 # 수정-1
contexts:
  - context:
      cluster: kube-c1 # 수정-1
      namespace: pickeatup-prod
      user: c1-admin # 수정-2
    name: kubernetes-c1-admin@kubernetes # 수정-3
current-context: kubernetes-c1-admin@kubernetes # 수정-3
kind: Config
preferences: {}
users:
  - name: c1-admin # 수정-2
    user:
      client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0F
```

주석이 붙은 번호들은 서로 같은 값을 가져야 한다.

## download cluster 2 config

```
scp c2-master:/etc/kubernetes/admin.conf ~/.kube/c2-config
```

위 설명처럼 수정한다.

## 두설정 파일을 병합하자.

KUBECONFIG 환경 변수를 생성하자.

```bash
export KUBECONFIG=~/.kube/c1-config:~/.kube/c2-config
```

## 병합된 설정을 확인하자.

```
kubectl config view
```

## 원하는 컨텍스트를 확인하자.

```bash
kubectl config use-context kubernetes-c1-admin@kubernetes # context name
kubectl config use-context kubernetes-c2-admin@kubernetes # context name
```

## kubeconfig 파일을 병합할 때에 kubectl에서 사용하는 규칙

- --kubeconfig 플래그를 설정했으면, 지정한 파일만 사용한다. 병합하지 않는다. 이 플래그는 오직 한 개 인스턴스만 허용한다.

- 그렇지 않고, KUBECONFIG 환경 변수를 설정하였다면 병합해야 하는 파일의 목록으로 사용한다. KUBECONFIG 환경 변수의 나열된 파일은 다음 규칙에 따라 병합한다.

  - 빈 파일명은 무시한다.
  - 역 직렬화 불가한 파일 내용에 대해서 오류를 일으킨다.
  - 특정 값이나 맵 키를 설정한 첫 번째 파일을 우선한다.
  - 값이나 맵 키를 변경하지 않는다. 예: 현재 컨텍스트를 설정할 첫 번째 파일의 컨택스트를 유지한다. 예: 두 파일이 red-user를 지정했다면, 첫 번째 파일의 - red-user 값만을 사용한다. 두 번째 파일의 red-user 하위에 충돌하지 않는 항목이 있어도 버린다.
  - KUBECONFIG 환경 변수 설정의 예로, KUBECONFIG 환경 변수 설정를 참조한다.

- 그렇지 않다면, 병합하지 않고 기본 kubeconfig 파일인 \$HOME/.kube/config를 사용한다.

이런 규칙으로 병합이 된다.

## .kube/config

파일 규칙을 이해하고 위에서 받은 두개의 파일을 직접 수정해서 합쳐도 된다. 이때 환경변수는 지워줘야한다.

```
unset KUBECONFIG
```

## 재시작시에도 항상 적용

vi .zshrc

```
export KUBECONFIG=~/.kube/c1-config:~/.kube/c2-config
```
