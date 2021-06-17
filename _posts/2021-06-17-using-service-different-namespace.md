---
layout: post
title: '다른 namespace에 있는 서비스 사용하기'
author: teamsmiley
date: 2021-06-17
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# 다른 namespace에 있는 서비스 사용하기

kubernetes 에서 프로젝트가 10개가 넘어가는데 모든 서비스에서 rabbitmq를 사용중이다 보니 똑같은 pod가 여러개 올라간다.

aws의 pod제한에 따라서 새 노드를 올려야하는 문제가 생겼다.

그래서 shared라는 namespace를 만들고 거기에 rabbmitmq하나만 올리고 모든 프로젝트에서 이걸 공유하는것으로 구조를 바꿨다.

## 작업

기존에는 rabbitmq라는 같은 namespace에 있는걸 사용했지만 지우고 shared에 있는걸 사용해야한다. 그러면 프로그램으로는 어떻게 해야할가.

간단하도 hostname을 rabbitmq에서 rabbitmq.shared라고 하면된다.

```txt
serviceName.namespace

rabbitmq.shared
```

이렇게 사용하면 된다.

## rabbitmq que vs vhost

vhost에 대해서 고민을 하엿으나 그냥 que를 여러개 만들어서 그냥 사용하기로 햇다.

나중에 논리적으로 나누고 싶으면 vhost를 만들어서 나눠주기로 햇다.
