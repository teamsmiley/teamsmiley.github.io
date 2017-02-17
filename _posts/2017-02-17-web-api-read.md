--- 
layout: post 
title: "Restful API 읽을거리" 
date: 2017-02-17 00:00  
author: teamsmiley 
tags: [api]
image: /files/covers/blog.jpg
category: {읽을거리}
---

# Restful API 읽을거리

<http://language.is/163> 

* Stateless

Stateless 하다는 것은 서버가 어떠한 Client 의 Status 도 저장하지 않는다는 뜻입니다. 따라서 Client 는 매 요청마다 자신을 인증할 수 있는 Token 이나 Key 등을 Request 에 포함시켜 전송합니다. 만약 클라이언트의 요청이 Stateful 하다면 서버가 클라이언트의 현재 상태를 저장해야하고, 클라이언트의 상태는 해당 서버에 종속이 됩니다. 만약 대규모 환경에서 동일한 웹서버를 다수 배치해 로드밸런싱을 하는 경우에 각 서버가 클라이언트의 상태, 세션을 공유할 수 있는 Redis 같은 별도의 시스템이 필요합니다. 그러나 Stateless 하다면 클라이언트의 요청은 어느 서버가 처리하던지 동일하게 처리할 수 있습니다. 클라이언트의 상태는 클라이언트의 요청 안에 모두 들어있으니까요.

* HATEOAS

REST의 영광을 향한 단계들 steps toward the glory of REST - 마틴 파울러

<http://jinson.tistory.com/190>


* OAUTH 2.0 – OPEN API 인증을 위한 만능 도구상자
<http://earlybird.kr/1584>

<http://bcho.tistory.com/914>

* [Spring HATEOAS]

<http://woniperstory.tistory.com/219>

* [읽어야함.]

<http://www.asp.net/web-api/overview/security/individual-accounts-in-web-api>

* RESTful Web Services: The Basics
<http://huewu.blog.me/110101220171>

<http://huewu.blog.me/110101273563>

* 실용적인 REST 이야기

<http://huewu.blog.me/110102472786>

<http://huewu.blog.me/110102524575>


* 당신의 API가 RESTFUL 하지 않은 5가지 증거
<https://beyondj2ee.wordpress.com/2013/03/21/당신의-api가-restful-하지-않은-5가지-증거/>




