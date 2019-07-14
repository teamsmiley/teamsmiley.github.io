---
layout: post
title: 'kubernetes mac에서 관리하기' 
author: teamsmiley
date: 2019-07-13
tags: [devops]
image: /files/covers/blog.jpg
category: {business}
---
# kubernetes mac에서 관리하기

## kubernetes cli 를 설치한다. 

```
brew install kubernetes-cli
```

## zsh 플러그인들 설치한다. 
```bash
cd ~/.oh-my-zsh/plugins/
# zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git\necho 
"source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

터미널 재시작 

## kubernetes 연결하기

```bash
scp root@master:/root/.kube/config ~/.kube/config
# 확인
kubectl get pods 
```

## dashboard 사용
* 설치 
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
```

* user 생성 
<https://github.com/kubernetes/dashboard/wiki/Creating-sample-user>

vi kube-admin-user.yaml
```yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```
* 적용
```
kubectl apply -f kube-admin-user.yaml
```

* 생성된 토큰을 찾아서 복사해두자.

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

생성이 잘못된 경우 전체 유저를 보여준다. 

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXpnNmZ3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwNzNmMTc1Yi1hZWZhLTQ4YWMtYTJiYS03M2RkM2ZmMDE3YTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.NImcwgUUCVj_FfB_Nzfv5rP91EIxlHDwOHPRolyNd5t-x6teH4SQTEb-4bOrk-ILtGvbDF7Nl4c6wqcO6pcujB81eG5t5F-PYH_0sQ3PF1gs3W1Eq_ZazrGi3nfqxajJcPARu7m2RgrUhoSMLaRKn2_6Lfadc4gV59AWU9lXnbQbpaarUgTDWBLCd8NGCG92X5IIS23aBGQDSXTkFQfOwI83fPeDJcwWhXs5yu-LHXHKQe7sIYV5tvQWJTyt_V2XZxjiAOW5pgiMPX8FWhZqmVbLbGtIMoFQeWi0CGFrQNoLb0X2Lb53wvSmKlFl051ItlLTyNqDuB7gO3DXYAydHQ

* 랩탑과 서버 연결

```bash
kubectl proxy
```

웹사이트로 확인하자.

<http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/>



