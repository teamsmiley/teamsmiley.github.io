---
layout: post
title: 'Visual-Studio에서-VI를-써보자' 
author: teamsmiley 
date: 2016-06-23
tags: [visual studio]
image: /files/covers/blog.jpg
category: {programe}
---

# Visual-Studio에서-VI를-써보자

요즘 마우스를 사용하지않고 코딩하는것에 매진하고 있다.

그중 하나로 코딩시 vi를 사용해보려고 한다..

## vsvim extension 설치

[vsvim_link] 에서 설치한다.

## enable/disable
enable/disable을 하고 싶으면 ctrl+shift+f12 이다.

가끔 이게 안되는경우에는 vs ==>tool==> option 에서

keyboard에 가서 초기화를 한번 시킨후

vsvim에 가서  Handle all with VsVim 을 클릭하고 OK 클릭

이후에 단축키를 확인해보자.

## 검색시 붙여넣기 사용 

vsvim 은 / 로 검색을 할때 붙여넣기가 안되서 불편하다.

그럴때는 /  다음에 home키를 한번 누른후 ctrl + v를 하면 복사가 가능하다.

 ## 검색시 대소문자 구분 하지 않는다.

vim에서 검색시 대소문자를 구분하고 싶지 않다.

:set ignorecase

하면 된다.

그런데 vs를 실행할때마다 해줘야한다.

불편함으로 세팅파일을 만들어서 매번 로딩시 자동으로 되게 해주자.

:set vimrcpaths? 해보자.

vimrcpaths=”C:\Users\Administrator;C:\Users\Administrator\vimfiles;C:\Users\Administrator”

이런 결과가 나온다. 
이중에 하나에 .vimrc라는 파일을 만들고 세팅을 넣어주면 vs가 자동 로딩한다는 이야기 

나는 C:\Users\Administrator\.vimrc 이경로를 사용

### 탐색기 이용 
탐색기에서 C:\Users\Administrator 폴더에 vimrc파일을 만든다. 

cmd 를 오픈해서 다음처럼 한다.

cd C:\Users\Administrator

move vimrc .vimrc

이제 파일이 생겼다.

### cmd에서 처리 

copy /b NUL C:\Users\Administrator\.vimrc

이제 메모장으로 파일을 열어서 다음처럼 작성한다.

:set ignorecase

![]({{ site.baseurl }}/assets/vim-ignorecase.png)

vs를 재시작

완료 

[vsvim_link]: https://visualstudiogallery.msdn.microsoft.com/59ca71b3-a4a3-46ca-8fe1-0e90e3f79329 "VS VIM EXTENTION"

