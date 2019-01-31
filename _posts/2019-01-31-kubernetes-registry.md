---
layout: post
title: 'kubernetes private registry' 
author: teamsmiley
date: 2019-01-31
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes 에서 private registry 사용 (ImagePullError)

노드에 접속해서 docker-login을 하고 터미널에서 docker pull을 하면 이미지를 가져온다.

그런데 스피네커를 통해서 디플로이만 하면 ImagePullError 에러 발생 

이런 경우는 private registry를 사용시 네임 스페이스마다 secret을 생성해줘야한다. 

```bash
kubectl create secret docker-registry <secret-name> \
--docker-server=<your-registry-server> \
--docker-username=<your-name> \
--docker-password=<your-pword> \
--docker-email=<your-email> \
--namespace=<namespace-name> 
```

이렇게 하면 스피네커가 배포하고 나면 쿠베가 레지스트리에서 잘 가져온다. 