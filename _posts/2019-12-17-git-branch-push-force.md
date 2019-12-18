---
layout: post
title: 'Git Branch Push Force' 
author: teamsmiley
date: 2019-12-17
tags: [git,terminal]
image: /files/covers/blog.jpg
category: {git}
---
# git 문제 해결

현재 dev브랜치가 있고 new-dev 브랜치가 있다. new-dev브랜치를 dev에 덮어 써야하는 상황이 발생 

```bash
git checkout new-dev
git branch -D dev # dev를 삭제
git checkout -b dev # dev를 생성
git push --set-upstream origin dev --force # 서버에 있는 dev에 강제로 이 버전을 푸시.
```

force는 쓰지 말아야하는데 어쩔수 없었음..

참고로 로컬에 머지를 해버린 경우 아직 push는 안한경우.

이러면 일단 로컬에 dev를 지운다음 서버에서 가져오면 된다. 

```bash
git checkout dev
git branch -D dev2
git pull 
```



