---
layout: post
title: 'Kubernes Alias'
author: teamsmiley
date: 2020-04-20
tags: [kube]
image: /files/covers/blog.jpg
category: { kube }
---

kubernetes 에서는 명령어가 길다 단축키를 만들어보자.

zsh을 사용하므로 zshrc파일을 수정한다. bash를 사용하면 bashrc를 수정하면된다.

vi ~/.zshrc

```bash
source <(kubectl completion zsh)

alias k='kubectl'

alias ns='kubectl config set-context $(kubectl config current-context) --namespace'

alias nsv='kubectl config view | grep namespace:'
```

이렇게 하면 kubectl을 다 치지않고도 명령어를 넣을수 있다.

oh-my-zsh을 사용하고 kube플러그인을 사용하면 기본적으로 적용이 되있다고 한다.

<https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl>

기타 많은 alias가 있으므로 참고해서 사용하면 될듯 싶다.

ns 와 nsv 대신에 플러그인에서 제공하는

```bash
kgns  #kubectl get namespaces	List the current namespaces in a cluster 전체 네임스페이스
kcgc  #kubectl config get-contexts context 확인
kcn   #kubectl config set-context context 설정
```

위 커맨드를 사용하면 된다.
