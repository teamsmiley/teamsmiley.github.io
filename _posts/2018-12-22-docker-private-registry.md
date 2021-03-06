---
layout: post
title: 'Docker Private Registry + SSL + Lets Encrypt + Username + Password' 
author: teamsmiley
date: 2018-12-22
tags: [devops]
image: /files/covers/blog.jpg
category: {linux}
---

# docker private registry 

centos 7  설치 

도커 설치 

## 폴더 생성 

```
mkdir -p /data/docker/auth
cd /data/docker
```

## lets encrypt ssl 발급받기 

```bash
sudo bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo yum update
sudo yum install epel-release git -y
sudo yum install python-pip -y
sudo yum install python-virtualenv -y

sudo pip install requests urllib3 pyOpenSSL --force --upgrade

cd /tmp
git clone https://github.com/certbot/certbot.git 
cd certbot

./certbot-auto certonly \
--manual \
--preferred-challenges=dns \
--email teamsmiley@gmail.com \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--debug \
--no-bootstrap \
-d registry.publishapi.com
```

_acme-challenge.registry txt 도메인에 등록하라고 나옴

```
Please deploy a DNS TXT record under the name
_acme-challenge.UR-DOMAIN.COM with the following value:

h1vJeUEv6AYJu5stnwlLy-xxx

Before continuing, verify the record is deployed.
```

도메인에 txt 레코드 등록하고 조금기다려서 dns가 업데이트가 되고 나면  엔터 

```
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/UR-DOMAIN.COM/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/UR-DOMAIN.COM/privkey.pem
   Your cert will expire on 2019-03-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

발급 됬음 

## 아이디 비번 발급

```bash
docker run \
  --entrypoint htpasswd \
  registry -Bbn USERNAME PASSWORD > /data/docker/auth/htpasswd
```


## 도커 실행
cd /data/docker/registry/
vi docker-compose.yml

```yml
---
version: "3.3"
services:
  registry:
    container_name: 'registry'
    restart: always
    image: registry
    privileged: true
    ports:
      - 5000:5000
    environment:
      TZ: "America/Los_Angeles"
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data/registry
      REGISTRY_HTTP_TLS_CERTIFICATE: /etc/letsencrypt/live/UR_DOMAIN/fullchain.pem
      REGISTRY_HTTP_TLS_KEY: /etc/letsencrypt/live/UR_DOMAIN/privkey.pem
    volumes:
      - /data/registry:/data/registry/docker/registry
      - /etc/letsencrypt:/etc/letsencrypt
      - ./auth:/auth
```

```
cd /data/docker/
docker-compose up -d
```

## 확인
```
docker login UR_DOMAIN:5000
```

이걸 안하면 spinnaker에서 docker image list를 못가져온다. 



