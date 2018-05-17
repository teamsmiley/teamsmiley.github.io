---
layout: post
title: 'IIS Web Deploy' 
author: teamsmiley 
date: 2016-08-12
tags: [IIS]
image: /files/covers/blog.jpg
category: {programe}
---

## Web Management Service Install 

Server manager -> Roles -> WebServer (IIS) -> InternetInformationServer (IIS) Manager ....

## Web Deploy 3.6 for Hosting Servers 설치 

Web Platform Install를 실행해서 Web Deploy 3.6 for Hosting Servers을 설치한다. 

![]({{ site.baseurl }}/assets/web_deploy_2.png)
## iis web deploy  설정 

* web deploy 3.6 다운로드 및 설치 <https://www.microsoft.com/en-us/download/details.aspx?id=43717>

* 설치 전 

![]({{ site.baseurl }}/assets/web_deploy_1.png)

* 설치 후 

![]({{ site.baseurl }}/assets/web_deploy_3.png)

* 관리자 서비스 실행 

![]({{ site.baseurl }}/assets/web_deploy_4.png)

* 설정 확인 

![]({{ site.baseurl }}/assets/web_deploy_5.png)

* 웹사이트를 선택하고 iis manager permissions를 선택한다. 

![]({{ site.baseurl }}/assets/web_deploy_6.png)

* 유저를 추가해준다  - 간단하게 어드민으로 했다.

![]({{ site.baseurl }}/assets/web_deploy_7.png)

서버는 완료 

* 이제 visual studio에서 배포 하면 된다.

게시방법 - web deploy

서버 - http://your.website.com:8172/MsDeploy.axd

![]({{ site.baseurl }}/assets/web_deploy_8.png)

완료

