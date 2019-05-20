---
layout: post
title: 'ingress nginx ssl' 
author: teamsmiley
date: 2019-02-07
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes ingress nginx with ssl (let's encrypt)

ingress nginx에 ssl을 추가해보자.

먼저 master에서 ssl을 만들어야한다. let's encrypt를 이용하여 ssl을 만들자. 

## let's encrypt 

```bash
# domain 셋업
UR_DOMAIN=aaa.com
UR-EMAIL=support@aaa.com

sudo yum update
sudo yum install epel-release git -y
sudo yum install python-pip -y
sudo yum install python-virtualenv -y

sudo pip install requests urllib3 pyOpenSSL --force --upgrade

yum install certonly

certbot certonly \
--manual \
--preferred-challenges=dns \
--email ${UR-EMAIL} \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--debug \
--no-bootstrap \
-d ${UR_DOMAIN}
```

_acme-challenge txt 도메인에 등록하라고 나옴

```
Please deploy a DNS TXT record under the name
_acme-challenge.UR_DOMAIN with the following value:

h1vJeUEv6AYJu5stnwlLy-xxx

Before continuing, verify the record is deployed.
```

도메인에 txt 레코드 등록하고 조금 기다린후 dns가 업데이트가 되면 커맨드에서 엔터 

```
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/UR_DOMAIN/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/UR_DOMAIN/privkey.pem
   Your cert will expire on 2019-03-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

발급 됬음 

## kube secret등록

```bash
CERT_NAME=AAA
UR_NAMESPACE=AAA
KEY_FILE=/etc/letsencrypt/live/${UR_DOMAIN}/privkey.pem
CERT_FILE=/etc/letsencrypt/live/${UR_DOMAIN}/fullchain.pem

kubectl create secret tls ${CERT_NAME} --key ${KEY_FILE} --cert ${CERT_FILE} -n ${UR_NAMESPACE} #인그레스 네임 스페이스를 꼭 넣어주자.
```

## ingress를 만들자. 

```yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prod
  namespace: prod
  annotations:
    kubernetes.io/ingress.class: "nginx"
    kubernetes.io/tls-acme: "true"
    kubernetes.io/ingress.allow-http: "false"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts: 
      - UR_DOMAIN
      secretName: CERT_NAME
    - hosts:
      - UR_DOMAIN2
      secretName: CERT_NAME2

  rules:
  - host: UR_DOMAIN
    http:
      paths:
      - backend:
          serviceName: echo-service
          servicePort: 80
  - host: UR_DOMAIN2
    http:
      paths:
      - backend:
          serviceName: echo-service
          servicePort: 80
```

## kubernetes에 적용한다.

``` 
kubectl apply -f ingress.yml
```

## 와일드카드 도메인 발급 

<https://teamsmiley.github.io/2019/02/07/lets-encrypt-ssl/> 참고해서 발급하면된다.

## 이제 ingress에 적용해보자. 

```yml
spec:
  tls:
    - secretName: UR_DOMAIN
      hosts:
        - "*.UR_DOMAIN"

rules:
  - host: "*.UR_DOMAIN"
    http:
      paths:
      - backend:
          serviceName: UR_SERVICE
          servicePort: 80
```

양쪽에 따옴표를 붙이는것이 중요하다.

`*.aaa.com`은 되지만 `auth.*.aaa.com`은 안된다.  도메인에서 설정도 안됨.

`*`는 항상 왼쪽 첫번째에 나와야한다. 






