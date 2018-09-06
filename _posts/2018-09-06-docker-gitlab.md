---
layout: post
title: 'GitLab on Docker' 
author: teamsmiley
date: 2018-09-06
tags: [gitlab,docker]
image: /files/covers/blog.jpg
category: {gitlab,docker}
---

# GitLab을 도커로 설치해보자. 

centos 7에  도커 최신버전을 설치하고 다음을 진행한다. 

참고 <https://docs.gitlab.com/omnibus/docker/>

## 설치및 실행 - docker run

```
sudo docker run --detach \
    --hostname gitlab.xgridcolo.com \
    --publish 443:443 --publish 80:80 --publish 30022:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

--publish 30022:22  ==> 이부분이 22면 기존 ssh와 포트 충돌 에러가 난다. 30022로 변경했음

위 커맨드를 실행하면 최신 깃랩 이미지를 받아서 실행을 한다. 

http://gitlab.xgridcolo.com로 접속을 해본다. 에러가 나면 다음을 확인하자.

* 외부 방화벽
* 서버 소프트웨어 방화벽 
* dns설정이 정확한가?

## docker compose로 다시 

```
mkdir /data/docker/gitlab
cd /data/docker/gitlab
vi docker-compose.yml
```

```yml
---
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.xgridcolo.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://gitlab.xgridcolo.com'
      # Add any other gitlab.rb configuration here, each on its own line
      gitlab_rails['gitlab_shell_ssh_port'] = 30022
  ports:
    - '80:80'
    - '443:443'
    - '30022:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```

## run

```bash
cd /data/docker/gitlab
docker-compose up -d
```

웹사이트 접속이 안되면 다음처럼 한다.

```
docker exec gitlab_web_1 update-permissions
docker restart gitlab_web_1
docker logs -f gitlab_web_1
```

꼭 docker logs -f gitlab_web_1 이 명령어로 로그를 확인하기 바란다.

## gitlab email을 보낼수가 있다. 

```bash
sudo docker exec -it gitlab_web_1 /bin/bash
vi etc/gitlab/gitlab.rb
```

```rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "your-email"
gitlab_rails['smtp_password'] = "your-password"
gitlab_rails['smtp_domain'] = "smtp.gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'peer' # Can be: 'none', 'peer', 'client_once', 'fail_if_no_peer_cert', see http://api.rubyonrails.org/classes/ActionMailer/Base.html
```

이제 회원가입을 하면 이메일이 발송된다.

## 초기화면에서 회원가입 없애기 

초기 화면에서 회원가입이 있으면 회사 내부만 쓰려고 하던 목적에 맞지 않는다. 없애보자.

* <http://gitlab.xgridcolo.com/admin/application_settings>

로그인 >> admin area >> Sign-up restrictions >> Sign-up enabled 을 off하면된다.

## 유저 추가 

<http://gitlab.xgridcolo.com/admin/users/new> 에서 유저를 추가한다. 그리고 유저 리스트로 가면 설정버튼이 보인다. 

설정버튼을 눌러서 들어가면 비밀번호를 세팅할수 있다. 

이제 사용자에게 알려줘서 접속하라고 하면된다.


