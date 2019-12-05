---
layout: post
title: 'kubernetes file upload' 
author: teamsmiley
date: 2019-12-05
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes 에서 file upload api를 만들고 난후 문제

dotnet core 로 file upload api를 애저에 쓰게 해서 만들었다. 로컬에서는 잘되었다. 

그런데 kube에 올리니 잘되는것처럼 보엿으니 1메가 이상(정확하지는 않음)부터 에러(413)가 나기 시작했다.

처리 해보자. 

1. azure가 1메가 이상을 기본으로 안받아주는가.=> 로컬에서 azure로 바로 붙여서 하면 잘된다. 심지어 애저로 안하고 하드디스크에 저장하게 해서 테스트 로컬은 잘됨 서버에서는 에러 azure문제가 아님 
2. dotnet core가 안받아주는가? kestrel이 잘 안해주는가? => 로컬에서는 잘됨 틀림 kestrel문제가 아닌거같은데.
3. kubernetes ingress를 의심해서 에러 메세지(413)와 함게 검색 비슷한 현상을 찾을수 있었다.

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: api
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 500m #추가
  namespace: stanley-dev
  labels:
    app: api
      protocol: TCP
  selector:
    app: api
```

위 코드처럼 수정을 하니 업로드가 잘된다. 



