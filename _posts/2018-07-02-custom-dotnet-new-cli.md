---
layout: post
title: 'dotnet cli를 커스터마이즈하기' 
author: teamsmiley 
date: 2018-07-04
tags: [regex]
image: /files/covers/blog.jpg
category: {program}
---

# custom dotnet new 만들기 

프로그램을 만들어주는 프로그램 만들기.

dotnet new 해서 반복되는 코드를 자동으로 생성해서 프로젝트에 넣어주면 좋겠음. 반복 작업을 많이 줄일수 있음

## 기본 기능 

### 템플릿 만들기
폴더 생성 
```bash
mkdir -p C:\Users\<your-id>\Desktop\smiley-template
cd C:\Users\<your-id>\Desktop\smiley-template
```

aaa.cs를 생성 - 이 파일이 나중에 생성될 파일이다. 

```
mkdir -p C:\Users\<your-id>\Desktop\smiley-template\.template.config
cd C:\Users\<your-id>\Desktop\smiley-template\.template.config
```
설정 파일 template.json 생성 
```json
{
  "$schema": "http://json.schemastore.org/template",
  "author": "Brian Kim",
  "classifications": [ "SmileyTemplate" ], 
  "name": "Smmiley Template Sample",
  "identity": "Teamsmiley.template.sample.CSharp",         
  "groupIdentity":"Teamsmiley.template",
  "preferNameDirectory":"true",
  "shortName": "smiley",                     // 이걸 사용해서 템플릿을 실행함
  "tags": {
    "language": "C#",                         // Specify that this template is in C#.
    "type":"item"
  },
  "sourceName": "Smiley.ProjectName",              // 커맨드라인에서 -n으로 해서 옵션을 주는데 그걸로 대치된다. 
  "preferNameDirectory":true
}
```

### 템플릿 설치하기 
```bash
dotnet new --install C:\Users\<your-id>\Desktop\smiley-template
```
결과가 다음처럼 보인다 smiley가 포함된걸 알수 있다. 
```
Console Application                               console          [C#], F#, VB      Common/Console
Class library                                     classlib         [C#], F#, VB      Common/Library
Contoso Sample 01                                 smiley           [C#]              Console
Unit Test Project                                 mstest           [C#], F#, VB      Test/MSTest
```


### 템플릿 실행하기
```bash
mkdir -p C:\Users\<your-id>\Desktop\newfolder
cd C:\Users\<your-id>\Desktop\newfolder
dotnet new smiley
```
폴더에 가보면 aaa.cs가 들어잇는걸 확인할수 있다. 

### 템플릿 삭제하기

```bash
dotnet new --uninstall C:\Users\<your-id>\Desktop\smiley-template
```

## 고급기능

## 디렉토리 파일 추가 

내가 원하는 디렉토리를 생성한후 파일도 생성하고 다시 템플릿을 설치하고 테스트해보자.  잘 된다. 

## 파일 내용을 변수로 받아서 바꿔주고 싶다.

예를들어 파일 내용에   -_____ 이 있는데  이걸 Smiley 로 바꿔주고 싶다.  symbols 을 이용하자.

template.json  파일에 다음을 수정하자.

```json
"symbols":{
  "modelName": { //이건 command line에 나오는 내용이다. 
    "type": "parameter",
    "defaultValue": "Smiley",
    "replaces":"-_____"
  }
}
```

이렇게 하고 템플릿 재설치 하고 실행해보자  
dotnet new smiley 

"-_____" 이런 내용이 있는 파일이 John Smith 로 바뀐다. (기본값을 사용)

modelName 은 템플릿 설치후 -h로 보면 보이는 화면이다.

```bash
dotnet new smiley -h

Contoso Sample 01 (C#)
Author: Contoso
Options:
  -m|--modelName
      string - Optional
      Default: John Smith
```

Smiley가 아니라 "Smiley"로 바꾸고 싶으면 다음처럼 한다. 

dotnet new smiley -m Smiley

잘 바뀐것을 알수 있다.

## 특정 값을 받아서 true일경우에만 넣고 싶은경우

json에 추가 
```json
"symbols":{
  "EnableContactPage":{
    "type": "parameter",
    "dataType":"bool",
    "defaultValue": "false"
  }
```

소스파일에 다음처럼 추가한다. 

```cs
#if (EnableContactPage)
public IActionResult Contact()
{
    ViewData["Message"] = "Your contact page.";

    return View();
}
#endif
```

이렇게 하면 생성하는 파일을 if로 조정할수 있다.  물론 커맨드로 값을 바꿀수 있다. 

## symbols규칙을 적용하기 싫은 페이지는 어떻게?

그런데 특정 페이지만  적용하고 싶지 않다..이런경우가 있을수 있음. 

이럴때는 source를 이용한다.  json에 다음을 추가한다. 

```json
"symbols": {
  ...
},
"sources": [
  {
    "modifiers": [
      {
        "condition": "(!EnableContactPage)",
        "exclude": [ "Views/Home/Contact.cshtml" ]
      }
    ]
  }
]
```

저 페이지는 제외한다. 

## cshtml 파일 
```
@*#if (EnableContactPage)
  <li><a asp-area="" asp-controller="Home" asp-action="Contact">Contact</a></li>
#endif*@
```

이렇게 사용하면된다.

## filename 을 바꿔보자. 

바꾸고 싶은 파일이름을 -_____ 으로 만들었다. 

json파일을 수정한다. symbol에 넣으면되고 위에 예시과 같으나 filerename만 다르다. 

```json
"FileModelName": {
  "type": "parameter",
  "dataType": "string",
  "fileRename": "-_____"
}
```

실행해보면 파일명이 바뀌었음을 알수있다. 

-_____.cs => Model.cs 이런식






