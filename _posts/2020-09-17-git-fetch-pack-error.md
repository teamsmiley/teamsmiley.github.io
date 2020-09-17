---
layout: post
title: "fatal: git fetch-pack: expected shallow list"
author: teamsmiley
date: 2020-09-17
tags: [angular, ionic]
image: /files/covers/blog.jpg
category: { programing }
---

# fatal: git fetch-pack: expected shallow list

오늘 gitlab runner를 추가햇는데 하나의 서버에서 위 에러를 발생

확인해보니 centos 7이 git 1.8.3.1 을 사용하는데 이버전은 fetch-pack을 지원하지 않음.

그래서 centos 7에서 git을 최신버전으로 업그레이드하면 해결됨.

```bash
git --version

> git version 1.8.3.1

yum remove git

yum -y install https://packages.endpoint.com/rhel/7/os/x86_64/endpoint-repo-1.7-1.x86_64.rpm

yum install git -y

git --version

> git version 2.24.1
```

gitlab runner가 문제없이 동작함.
