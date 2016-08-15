---
layout: post
title: 'VS Code Default Setting 변경' 
author: teamsmiley 
date: 2016-08-15
tags: [program]
image: /files/covers/blog.jpg
category: {program}
---
# VS Code Default Setting 변경 

VS Code를 실행후 메뉴 >> Code >> Preference >> User Settings를 클릭하자.
단축키는 &#8984; + , 

에티터 화면에서 default Setting과 setting.json이 보인다. 

![]({{site_baseurl}}/assets/vs_settings01.png)

Default.Setting는 수정이 안된다. 수정을 하고 싶은 부분이 잇으면 복사해서 __setting.json__ 에 복사해 넣으면 된다.

json 형태이므로 꼭 설정 끝에 ,로 구분하고 마지막 라인에는 ,를 빼줘야한다.

수정해 보자.

## 마우스 휠로 폰트 변경 

```
"editor.mouseWheelZoom": true
```

저장후 확인해보자. 

VS Code를 재시작후 문서를 로딩한 후   Ctrl + 마우스 휠 을 올렸다 내렷다 해보자. 
폰트가 바뀌는것을 볼수가 있다. 

## 기본 폰트 크기 키우기 


노안이 와서 글자가 잘 안보이기 시작햇다. 폰트 사이즈가 12면 충분햇는데 이제는 조금 늘려야겠다.

매번 VS Code를 켜고 마우스를 이용해서 폰트를 변경햇는데 귀찮다...

```
    "editor.fontSize": 14,
    "editor.mouseWheelZoom": true
```
폰트 설정을 저장하자 마자 폰트가 바뀌는 것을 알수가 있다. 

## 파일을 열때 기존 VS Code를 사용하자.

기본 세팅은 파일을 열때마다 vs code가 열린다. vs code를 전체화면으로 해서 하나의 desktop을 사용하는 나로써는 상당히 불편하다. 

하나의 vs code에서 모든 파일이 탭으로 열리면 좋겠다. 

설정해보자. 

```
    "window.openFilesInNewWindow": false,
    "editor.fontSize": 14,
    "editor.mouseWheelZoom": true

```

## VS Code가 Full Screen에서 꺼졌을때 다시 실행하면 풀스크린으로 만들어주기 

나는 풀스크린으로 vs code를 사용하는데 매번 처음 켤때 전체 화면으로 가면 좋겟다.는 생각을 했엇다. 
옵션이 있다...오..~~ms 많이 좋아진듯..

```
    "window.restoreFullscreen":true,
    "window.openFilesInNewWindow": false,
    "editor.fontSize": 14,
    "editor.mouseWheelZoom": true 
```

꼭 전체화면으로 해두고 난 후 프로그램을 종료시켜야한다. 

단축키 &#8984; + Q 


일단 여기까지 하자. 



