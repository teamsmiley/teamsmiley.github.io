---
layout: post
title: 'Wsl2에서 gitlab wiki 사용하기' 
author: teamsmiley
date: 2020-04-28
tags: [kube]
image: /files/covers/blog.jpg
category: {wsl}
---

# wsl2에서 gitlab 위키 사용하기

## 관련 패키지 설치 
```bash
apt-get install ruby ruby-dev make zlib1g-dev libicu-dev build-essential git cmake pkg-config libssl-dev
gem install github-markdown gollum
```

## gitlab wiki를 클론한다.

```bash
git clone ssh://git@gitlab.XXX.com/XXX/XXXXX.wiki.git
cd XXXX.wiki
```

## gollum을 실행한다.
```bash
gullum

>[2020-05-04 11:09:00] INFO  WEBrick::HTTPServer#start: pid=22642 port=4567  << 포트번호.
```

웹으로 접속해서 확인한다.

http://localhost:4567


## 로컬에서 수정하자.

이제 로컬에서 vs code등으로 md파일을 수정할수 도 있고 웹을 통해 수정도 가능하다. 

그러나 로컬에서 md파일 수정의 경우 웹에 바로 업데이트가 안되는경우가 있다.

그 이유는 파일을 변경후 커밋을 안했기 때문이다 gollum은 꼭 커밋을 해야 웹페이지에서 수정사항이 보인다.

웹으로 수정을 하는 경우에는 자동으로 커밋이 된다. 

다 완료가 되면 git push를 하면 gitlab사이트에 위키가 업데이트가 된다.

## 매번 gollum을 시작하기 귀찮다.

wsl2가 systemd를 지원하지 않는거 같다. 아래 내용은 일단 대기 

```bash
cat<<EOF | sudo tee /etc/systemd/system/gollum.service
[Unit]
Description=Gollum wiki server
After=network.target
After=syslog.target

[Service]
Type=simple
User=gollum
Group=gollum
WorkingDirectory=/mnt/c/Users/ragon/Desktop/GitLab/ticket/ticket.wiki # your-path
ExecStart=/usr/local/bin/gollum --live-preview --config "/etc/gollum/config.rb"
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF
```
재시작하자. dos cmd에서 
```bash
wsl -t Ubuntu-20.04
```


