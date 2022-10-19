---
layout: post
title: 'Docker Volume Mount Permission'
author: teamsmiley
date: 2021-08-01
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# Docker Volume Mount시 Permission 관련 문제

오랜만에 docker-compose를 만들일이 있어서 file을 볼륨 마운트로 처리하였다.

그런데 뭐가 잘 안되서 이해가 안되서 하나씩 찾아보았다.

도커에 볼륨 마운트를 한 파일이 읽기 전용이라 호스트에서 아무리 바꾸어도 도커에서 그 파일을 확인해보면 도커 로딩시 읽었던 파일 그대로 되있다.

문제는 호스트 서버에서는 파일의 소유주가 ubuntu였고 755 로 권한이 주어져 있었다.

도커는 root로 돌고 있엇다. sudo 명령어를 사용하여 실행에는 문제가 없엇으나 실행된 도커는 파일에 쓰기 권한이 없었던 것이엿다.

해결방법은 파일의 소유주를 root로 모두 변경하고 도커를 올리면 호스트에서 수정된 파일이 도커안에서도 수정되었다.

마운트전 권한을 잘 확인하는 법도 필요하다.

## 추가

다시 해보니 잘 안됨.

파일 권한을 777로 하면 호스트에서 파일을 바꾸면 도커에도 잘 들어간다.

일단 이렇게 해결은 햇는데 이상함.

## 해결

드디어 해결했다.

`docker exec -it aaa bash` 로 들어가서 `id`를 치고 엔터해보면 동작하고 있는 유저 그룹등이 나온다.

호스트에서 `chown -R 123:123 XXXX`으로 유저를 위에서 확인된 유저로 지정을 해주면 된다.