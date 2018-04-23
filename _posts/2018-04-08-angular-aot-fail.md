---
layout: post
title: 'Angular dotnet core project Aot Fail' 
author: teamsmiley 
date: 2018-04-07
tags: [angular]
image: /files/covers/blog.jpg
category: {program}
---

# angular dotnet core project aot build시 실패

## 실패 만들기

angular 프로젝트에서 다음처럼 해보자.

```bash
dotnet publish -c Release -o app
```

## 이상하게 다음 에러를 내면서 성공이 안된다

```txt
Module not found: Error: Can't resolve './../$$_gendir/ClientApp/app/app.browser.module.ngfactory'
```

## 다음처럼 npm을 하나 설치해주자

```bash
npm install enhanced-resolve@3.3.0
```

## 재도전

```bash
dotnet publish -c Release -o app
```

성공한다.
