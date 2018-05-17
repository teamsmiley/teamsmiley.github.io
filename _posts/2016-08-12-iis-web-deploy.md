---
layout: post
title: 'IIS Web Deploy' 
author: teamsmiley 
date: 2016-08-12
tags: [IIS]
image: /files/covers/blog.jpg
category: {programe}
---

## iis를 구성요소와 같이 설치 한다.

## Server manager -> Roles -> WebServer (IIS) 


* Logging Tools
* Request Monitor
* .Net Extensibility
* ASP.Net 
* ISAPI Extensions
* ISAPI Filters, 
* basic Authentication
* request filtering
* iis management console
* iis management scripts and tools 
* Management Service.

## Web Platform Installer 설치

Web Platform Installer 5.0 최신 버전을 다운로드 받아 설치합니다.

## Web Deploy 3.6 for Hosting Servers 설치 

Web Platform Install를 실행해서 Web Deploy 3.6 for Hosting Servers을 설치한다. 

![]({{ site.baseurl }}/assets/web_deploy_900.png)

여러개의 참조프로젝트까지 다 설치가 된다. 

![]({{ site.baseurl }}/assets/web_deploy_901.png)


## 관리자 서비스 실행 

서버이름을 클릭하고 management service를 클릭한다.

![]({{ site.baseurl }}/assets/web_deploy_903.png)

설정을 하고 start를 한다. 

* 설정 확인 
  * windows credentials only로 해도 된다.
  * Enable remote connection을 on하고 시작 

![]({{ site.baseurl }}/assets/web_deploy_5.png)

## 서비스가 도는지 확인하고 자동시작으로 바꿔준다. 

![]({{ site.baseurl }}/assets/web_deploy_904.png)

## 로컬에서 테스트해보자.

꼭 https로 해야한다. ssl 에러가 나면 무시하고 진행을 한다. 

<https://localhost:8172/msdeploy.axd>

userid/password를 물어보면 성공 


* 이제 visual studio에서 배포 하면 된다.

게시방법 - web deploy

서버 - http://your.website.com:8172/MsDeploy.axd

![]({{ site.baseurl }}/assets/web_deploy_8.png)

완료

