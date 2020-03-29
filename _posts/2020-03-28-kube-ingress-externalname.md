---
layout: post
title: 'Kubernetes에서 외부 서비스에 연결하기' 
author: teamsmiley
date: 2020-03-28
tags: [kubernetes]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# Kubernetes에서 외부 서비스에 연결하기

기존에 있는 웹서비스를 kubernetes를 이용하여 외부에 오픈해야할 일이 생겼다.

인그레스를 설정하고 서비스를 ExternalName으로 설정하자. 

```yml
kind: Service
apiVersion: v1
metadata:
  name: proxy-google-com
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ExternalName
  externalName: www.google.com

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: proxy-to-google
spec:
  rules:
  - host: aaa.xgridcolo.com
    http:
      paths:
      - path: /
        backend:
          serviceName: proxy-google-com
          servicePort: 80
```

이제 aaa.xgridcolo.com 도메인세팅을 하고 호출해보면 구글로 가는것을 볼수 있다.

<https://github.com/kubernetes/ingress-nginx/pull/629#issue-222930691>


![]({{ site_baseurl }}/assets/2020-03-29-06-14-34.png)
