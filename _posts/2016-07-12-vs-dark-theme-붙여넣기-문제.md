---
layout: post
title: 'vs dark theme 붙여넣기 문제'
author: teamsmiley 
date: 2016-07-12 00:00
tags: [vs]
image: /files/covers/blog.jpg
category: {program}
---


개인적으로 프로그램을 하면서 동시에 기록하는걸 좋아한다.

구글 문서를 사용하면 이미지도 한꺼번에 넣을수 잇어서 메모장이나 마크다운에디터보다 좋아한다.

그런데 visual studio dark  theme을 사용하면 복사하여 구글문서에 붙여넣기를 하면 다음처럼 보인다.

![]({{ site.baseurl }}/assets/bad_paste.png)

매번 노트패드에 복사해서 다시 복사하곤 했다.

귀찮아서 이번에 한번 찾아봤다.

내가 기존부터 사용해오고 있었던 Productivity Power Tools 2015 에  기능이 있다.

https://visualstudiogallery.msdn.microsoft.com/34ebc6a2-2777-421d-8914-e29c1dfa7f5d 여기서 다운이 가능하고 vs 에서 검색해서 누겟으로 설치하면된다.

설치후 vs 재시작한다.

재시작후 tools >> options으로 간다.

다음 메뉴를 찾는다.

![]({{ site.baseurl }}/assets/productive_menu.png) 

다음처럼 설정을 바꾼다.

Change EmitSpanClass : True

Check EmitSpanStyle : True

BeforeCodeSnippet

기본
```html
<pre style=”{font-family}{font-size}{font-weight}{font-style}{color}{background}”>
```

수정후
```html
<style type=”text/css”>.identifier {color:black !important;}</style><pre style=”{font-family}{font-size}{font-weight}{font-style}”>
'''
완성후

 
![]({{ site.baseurl }}/assets/productive_menu_2.png) 


ok 를 누르고 다시 복사를 해보자…



![]({{ site.baseurl }}/assets/after_remove_background.png) 

복사가 잘된다…

컬러가 필요가 없다고 생각하시는분은…

적당히 위 옵션을 바꿔서 사용하면된다.

 
