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
apt-get install ruby ruby-dev make zlib1g-dev libicu-dev build-essential git cmake

gem install github-markdown
gem install gollum
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



