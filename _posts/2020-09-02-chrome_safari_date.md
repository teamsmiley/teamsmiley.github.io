---
layout: post
title: "datetime safari vs chrome"
author: teamsmiley
date: 2020-09-02
tags: [datetime]
image: /files/covers/blog.jpg
category: { program }
---

# safari and chrome datetime 처리 방법

앵귤러 datetime pipe가 크롬에서는 문제가 없는데 사파리에서 에러가 나서 확인해봄

## chrome

서버에서 `2020-01-01 00:00:00` 가 오면 로컬 시간으로 보여줌 문제는 서버가 utc/gmt일 경우 로컬 시간이랑 안맞음.

## safari

서버에서 `2020-01-01 00:00:00` 가 오면 utc로 가정하고 로컬 시간으로 변경해서 보여줌.

## 둘다 같은 값을 사용하기 위한 처리 방법

서버에서 `2020-01-01T00:00:00+00:00` 이것처럼 타임존을 붙여서 보내면 크롬/사파리가 같은 날/시간으로 인식을 한다.

C#에서 다음처럼 처리

```cs
Created.ToString("yyyy-MM-ddTHH:mm:sszzz")
```

이제 두개의 브라우저가 모두 같은 화면을 랜더링 한다.

서버에 모든 시간을 Utc로 사용하고 위 방법을 이용하면 문제없이 대부분의 문제가 없어질거같다.

## 주의

`2020-01-01T00:00:00+00:00`는 되지만 `2020-01-01 00:00:00+00:00`는 사파리에서 에러가 난다.
