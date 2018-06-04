---
layout: post
title: 'Jenkins 사용시 팁' 
author: teamsmiley 
date: 2018-06-02
tags: [jenkins github]
image: /files/covers/blog.jpg
category: {program}
---

# jenkins에서 사용시 팁

## github plugin 설치하기 

jenkins >> manage jenkins >> plugin manager 

* GitHub API Plugin
* GitHub Authentication plugin
* GitHub Branch Source Plugin
* GitHub Integration Plugin
* GitHub plugin

이렇게 설치하자.

## trigger 설정 

트리거중에 GitHub hook trigger for GITScm polling 이거를 선택해준다. 

## build 설정 

Execute Windows batch command 를 추가해서 빌드한다. 
```bash
cd console
call npm i 
call npm run build:ssr-staging 

xcopy /Y /S wwwroot c:\temp\console_root_staging

"C:\Program Files (x86)\IIS\Microsoft Web Deploy V3\msdeploy.exe" -verb:sync -source:IisApp='c:\temp\console_root_staging' -dest:iisapp='aaa.com',computerName='https://192.168.1.19:8172/msdeploy.axd?site=aaa.com',authType='basic',username='admin',password='password' -enableRule:AppOffline  -allowUntrusted 

rmdir /S /Q c:\temp\console_root_staging
```

## npm error시 다음으로 진행하지 않게

문제가 생겼다.  npm i 나 npm run build:ssr-staging을 할때 에러가 나더라도 계속 진행이 되버린다. 

npm 에서 에러가 나면 에러를 보고하고 멈추면 좋겠다. 

그래서 찾아봤더니 다음처럼 하면된다.

```bash 
cd console
call npm i || exit /b -1
call npm run build:ssr-staging || exit /b -1

xcopy /Y /S wwwroot c:\temp\console_root_staging

"C:\Program Files (x86)\IIS\Microsoft Web Deploy V3\msdeploy.exe" -verb:sync -source:IisApp='c:\temp\console_root_staging' -dest:iisapp='aaa.com',computerName='https://192.168.1.19:8172/msdeploy.axd?site=aaa.com',authType='basic',username='admin',password='password' -enableRule:AppOffline  -allowUntrusted 

rmdir /S /Q c:\temp\console_root_staging
```

|| exit /b -1 이것이 에러가 나면 바로 리턴을 해버린다. 


## 소스코드 커밋하면 자동 빌드하기 

github project >> setting >> integrations & service >> Add service 

jenkins로 검색해서 jenkins github service를 추가한다. 

Jenkins hook url 을 현재 젠킨스서버의 url을 사용한다. /github-webhook/를 추가하는것 잊지말자.

```
http://ci.aaa.com/github-webhook/
```

이제 됬다 소스코드가 커밋이 되면 깃허브가 ci서버를 호출을 해서 빌드를 시작하는거같다. 

실제로 잘 안되서 확인해보니 방화벽에 막혀있다 방화벽체크를 해서 깃허브가 ci서버 웹후크를 할수 있게 포트를 열어줘야한다.





