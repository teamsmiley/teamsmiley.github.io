---
layout: post
title: 'dotnet core 2 서버에서 log 보기' 
author: teamsmiley 
date: 2018-04-30
tags: [dotnet core]
image: /files/covers/blog.jpg
category: {program}
---

# aspnet core log 확인하기

## 배포시 logs폴더를 생성한다

project.csproj 파일을 수정한다. 

```xml
<Target Name="CreateLogsFolderDuringCliPublish" AfterTargets="AfterPublish">
  <MakeDir Directories="$(PublishDir)logs" Condition="!Exists('$(PublishDir)logs')" />
</Target>

<Target Name="CreateLogsFolderDuringVSPublish" AfterTargets="FileSystemPublish">
  <MakeDir Directories="$(PublishUrl)logs" Condition="!Exists('$(PublishUrl)logs')" />
</Target>
```

## 배포후 web.config를 수정한다.

<!-- <aspNetCore processPath="dotnet" arguments=".\rc-idp.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" /> -->
<aspNetCore processPath="dotnet" arguments=".\rc-idp.dll" stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" /> <!--true로 변경-->

## 앞에 포스트처럼 nlog를 설정하여 로그를 본다. 




