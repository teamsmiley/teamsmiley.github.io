---
layout: post
title: 'Kubernetes Log보기'
author: teamsmiley
date: 2020-05-10
tags: [Kubernetes]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# Kubernetes에서 로그 보기

기본적으로 kube에서는 다음처럼 로그를 보면된다.

```bash
kubectl get pod --all
kubectl logs -f  podxxxx
```

이러면 로그를 계속 볼수 있다. 그런데 너무 로그가 많으면 오래 걸린다. 특별시 시간을 줄수 있다.

```bash
kubectl logs -f xxxx  --since=5m
```

이제 5분전 로그부터 볼수 있다.

replica 세팅을 하면 같은 docker가 여러개 올라간다. 이런데 리퀘스트를 던지면 어떤 pod로 들어간지 알수가 없어서 불편하다.

전체 pod의 로그를 보고 싶다. (같은 docker 이미지.) deployment를 사용하면된다.

```bash
kubectl logs -f deployment/mobile-php --all-containers=true --since=5m
```

이러면 이제 mobile-php 로 생성된 모든 pod의 로그를 한꺼번에 보여준다.
