---
layout: post
title: 'kubernetes delete namespace' 
author: teamsmiley
date: 2019-09-28
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes delete namespace 

```
kubectl delete ns AAA
```

이러면 관련 리소스가 다 지워진다.

그런데 가끔 행이 걸리는 경우가 있다.

다음처럼 하면 된다.

on mac
```
brew update
brew install jq 
kubectl proxy &
```

터미널을 하다 더 띄워서 

```bash
NAMESPACE=delete-namespace
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```

kubectl proxy를 이용하는것이 핵심



