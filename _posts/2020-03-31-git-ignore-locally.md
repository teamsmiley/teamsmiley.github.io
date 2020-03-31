---
layout: post
title: 'git에서 특정파일을 임시로 ignore하기 ' 
author: teamsmiley
date: 2020-03-31
tags: [git]
image: /files/covers/blog.jpg
category: {git}
---

# git에서 특정파일을 임시로 ignore하기 

작업을 하다 보면 가끔 내 컴퓨터에만 적용이 되야하는경우가 있다. 그런경우에는 내 컴퓨터에는 파일이 변경되더라도 무시 되야한다.

파일의 변경 상태 무시

git update-index --assume-unchanged <file>

무시한 파일을 다시 트래킹 하기

git update-index --no-assume-unchanged <file>

무시 파일 목록

git ls-files -v | grep "^[[:lower:]]"

```bash
git update-index --assume-unchanged docker-env/www/www-php.ini
git update-index --no-assume-unchanged docker-env/www/www-php.ini
```
