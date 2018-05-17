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

* basic Authentication
* request filtering
* Logging Tools
* Request Monitor
* .Net Extensibility
* ASP.Net 
* ISAPI Extensions
* ISAPI Filters, 
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

![]({{ site.baseurl }}/assets/web_deploy_902.png)

* 설정 확인 
  * windows credentials only로 해도 된다.
  * Enable remote connection을 on하고 시작 

![]({{ site.baseurl }}/assets/web_deploy_5.png)


* 이제 visual studio에서 배포 하면 된다.

게시방법 - web deploy

서버 - http://your.website.com:8172/MsDeploy.axd

![]({{ site.baseurl }}/assets/web_deploy_8.png)

완료

