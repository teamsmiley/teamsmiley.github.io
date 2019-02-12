---
layout: post
title: 'lets encrypt with wildcard cert' 
author: teamsmiley
date: 2019-02-07
tags: [devops]
image: /files/covers/blog.jpg
category: {ssl}
---

# let's encrypt with wildcard cert

let's encrypt 인증서에 와일드카드 도메인을 적용해보자.

```bash
UR_DOMAIN=aaa.com

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
--email brian@rendercore.com \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--debug \
--no-bootstrap \
-d *.UR_DOMAIN # 이게 중요
```


_acme-challenge txt 도메인에 등록하라고 나옴

```
Please deploy a DNS TXT record under the name
_acme-challenge.UR-DOMAIN with the following value:

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
   /etc/letsencrypt/live/UR-DOMAIN/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/UR-DOMAIN/privkey.pem
   Your cert will expire on 2019-03-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

발급 위치 : /etc/letsencrypt/live/UR-DOMAIN/

확인

```bash
/tmp/certbot/certbot-auto certificates
```

*로 발급된걸 알수있다.

웹서버에 적용하면된다.

## 윈도우 
<https://github.com/PKISharp/win-acme/releases> 를 사용하자.


## 기타 사용법

```bash
sudo /tmp/certbot/certbot-auto renew --dry-run
sudo /tmp/certbot/certbot delete --cert-name example.com
```