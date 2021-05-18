---
layout: post
title: 'aws alb controller'
author: teamsmiley
date: 2021-05-17
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# aws alb controller

기존에 bearmetal kubernetes를 사용하다 eks를 쓰니 달라진점이 조금 있어서 정리해본다.

기존에는 ingress-nginx를 사용해서 트래픽을 분산햇는데 eks에서는 alb controller라는걸 지원해준다.

alb controller는 yaml에 이런저런 설정을 하면 자동으로 aws application load balance를 만들어준다.

써보니 편한거같다. aws는 하나도 몰라도 yml에 인그레스만 설정하면 다 해준다.

관련 내용은 여기를 참고 <https://github.com/kubernetes-sigs/aws-load-balancer-controller>

기본내용은 위 링크를 한번 보면되는데 내가 막힌 부분이 있어서 그것만 정리해보자.

## 기본 사용법

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: www
  namespace: www-staging
  annotations:
    kubernetes.io/ingress.class: 'alb' # alb를 사용하겠다
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]' # 포트를 80 443을 로드발란스에서 오픈하겟다.
    alb.ingress.kubernetes.io/scheme: internet-facing # 인터넷(public)에서 접속이 되게 한다.
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:849059793568:certificate/199b8e95 # load balance에 ssl을 적용
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
spec:
  rules:
    - host: www.aaa.com
      http:
        paths:
          - path: /*
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation
          - path: /*
            backend:
              serviceName: www
              servicePort: 80
```

이걸 사용하면 자동으로 aws application load balance도 만들어주고 listen port도 추가해준다. ssl도 적용해준다.

ssl의 경우 aws certificate manager 에서 미리 만들어 둬야한다.

## http를 https로 redirect

위에 샘플에 적혀는 있으나 특별히 따로 설명한다.

anotation에 다음 추가

```yml
alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
```

그리고 path에 다음 추가

```yml
- path: /*
  backend:
    serviceName: ssl-redirect
    servicePort: use-annotation
```

이러면 http로 접근하면 https로 리다이렉트를 시켜준다.

관련 내용은 여기를 참고하자. <https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/guide/tasks/ssl_redirect.md>

## ssl backend

특정 pod는 프로그램 자체에서 ssl로 접근을 받아야할 필요가 있을때 alb controller에서는 다음처럼 처리한다.

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: auth
  namespace: auth-staging
  annotations:
    kubernetes.io/ingress.class: 'alb'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-west-2:849059793568:certificate/199b8e95-fe5b-43e6-b499-061e4f133011
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
    alb.ingress.kubernetes.io/backend-protocol: HTTPS # 여기 추가
spec:
  rules:
    - host: www.aaa.com
      http:
        paths:
          - path: /*
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation
          - path: /*
            backend:
              serviceName: auth
              servicePort: 443
```

anotation에 다음 추가를 볼수 있다.

```yml
alb.ingress.kubernetes.io/backend-protocol: HTTPS # 여기 추가
```

만약 이걸 추가하지 않으면 이런 에러를 볼수가 있다.

```txt
Getting “Handshake failed…unexpected packet format”
```

alb가 기본적으로 http로 통신을 시도하므로 포트는 443을 쓰면서 http를 보내게 되다보니 이런 에러가 나온다.

## health check

로드 발란스가 기본적으로 pod를 다 체크해서 서비스를 유지해준다. 특별히 health check 경로를 수정하려면 다음처럼 하자.

```yml
alb.ingress.kubernetes.io/healthcheck-protocol: HTTPS #기본값 http
alb.ingress.kubernetes.io/healthcheck-path: /api/values
```

pod가 ssl을 기대하고 있으면 healthcheck-protocol도 맞는값을 넣어줘야한다.

오늘은 일단 여기까지.
