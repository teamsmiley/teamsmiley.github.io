---
layout: post
title: 'google api 사용법'
author: teamsmiley
date: 2021-06-25
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# google api 사용법

## 구글 API 콘솔에서 프로젝트 생성

![]({{ site_baseurl }}/assets/google-api/google-api-00.png)

## 구글 API 라이브러리 추가

![]({{ site_baseurl }}/assets/google-api/google-api-00-1.png)

![]({{ site_baseurl }}/assets/google-api/2021-06-25-09-27-27.png)

원하는 라이브러리 추가

![]({{ site_baseurl }}/assets/google-api/2021-06-25-09-28-11.png)

## 사용자 인증 정보 추가

![]({{ site_baseurl }}/assets/google-api/google-api-01.png)

![]({{ site_baseurl }}/assets/google-api/google-api-02.png)

![]({{ site_baseurl }}/assets/google-api/google-api-03.png)

![]({{ site_baseurl }}/assets/google-api/google-api-04.png)

## 프로젝트 삭제

사용이 끝나면 삭제

### 프로젝트 선택

원하는 프로젝트를 선택한다.

![]({{ site_baseurl }}/assets/google-api/google-api-00.png)

### 프로젝트 설정 클릭

![]({{ site_baseurl }}/assets/google-api/google-api-06.png)

### 종료 클릭

![]({{ site_baseurl }}/assets/google-api/google-api-07.png)

![]({{ site_baseurl }}/assets/google-api/google-api-08.png)

![]({{ site_baseurl }}/assets/google-api/google-api-09.png)

## api 제한

api를 제한하지 않으면 사실 남들이 apikey를 가져가서 다 사용해버리면 과금이 엄청 되버린다. 이부분을 항상 고려해야할듯 싶다.

일단 두개로 나눈다 외부에 오픈되는 키와 내부적으로만 사용되는키

### 외부에 오픈되는 키

외부에 오픈되는 키는 웹화면에서 소스보기를 하면 보이는 키이다. 이 키는 아무나 볼수가 있어서 누가 사용해버리면 과금이 발생한다.

이 키는 apikey에 제한을 꼭 걸어야한다.

- http 리퍼러를 통해 내 도메인에서만 들어오는 요청을 허용해야한다. 물론 개발을 위해서 localhost도 필수

- api 제한 사항도 꼭 걸어야한다. 필요한 api만 이 키에 허용을 해준다.

### 내부적으로 사용되는키

api에서 사용되는 키를 말한다 코드가 서버에 있으므로 브라우저에 오픈이 되지 않는다.

서버가 고정아이피라면 ip로 고정을 할수 있다. 이걸 추천한다.

그런데 저는 아마존이라 그냥 전체를 오픈해서 사용중이다. 어차피 다른누군가 알 수가 없으므로 큰 의미는없으나 api모니터링을 좀 해야할듯 싶다.

누가 아마존사용하면서 어플리케이션 제약에 서버 아이피로 처리가 가능하신분은 알려주시기 바란다.

api는 꼭 본인이 사용하는 api만 체크를 하기 바란다.

## 궁금한것

하다보니 뭔가 api가 기본으로 설치된게 너무 많다..뭐를 쓰는건지도 모르고 정리가 잘 안되는 느낌이다.

전체 프로젝트 리스트를 보면 구글 docs나 spredsheet에 쓰이는듯한 느낌적 느낌이 드는 프로젝트가 많이 있다 이게 뭘가?

![]({{ site_baseurl }}/assets/google-api/2021-06-25-09-38-01.png)
