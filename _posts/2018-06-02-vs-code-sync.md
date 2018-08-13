---
layout: post
title: 'VS Code Settings Sync' 
author: teamsmiley 
date: 2018-06-02
tags: [vs code]
image: /files/covers/blog.jpg
category: {program}
---

# VS Code Settings Sync를 이용하여 여러개 컴퓨터에서 동기화 해보자. 

## install setting sync

ctrl + shift + P enter

type install extension  enter 

search Settings Sync 

click install 

![]({{ site.baseurl }}/assets/sync-1.PNG)

또는 아래 링크에서 인스톨을 누르면된다. 

<https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync>

설치가 되면 

ctrl + shift + P 를 누르고 sync를 입력하자. 

그럼 다음그림이 나온다.

![]({{ site.baseurl }}/assets/sync-2.PNG)

일단 reset extension setting을 먼저 하자. (기존 정보가 있을지도 모르니 지우자.)


## gist token

github에 접속해서 token을 얻는다.

** Settings > Developer settings > Personal access tokens **

![]({{ site.baseurl }}/assets/sync-3.PNG)

이름 입력 ,  gist를 선택 후 완료 누르면 토큰을 얻을수 있다. 이건 다시 알아낼수가 없으므로 잘 저장해둔다. 

![]({{ site.baseurl }}/assets/sync-4.PNG)


## 현재 설정을 업로드후 gist id찾기 

이제 alt + shift + u를 누르자. 

![]({{ site.baseurl }}/assets/sync-5.PNG)

화면에 조금전에 받아둔 토큰을 넣은후 엔터

![]({{ site.baseurl }}/assets/sync-6.PNG)

플러그인이 Gist를 만든다.  아웃풋 화면을 보면 gistid를 볼수 있다. 

또는 본인아이디의 Gist화면으로 가보자. 

https://gist.github.com/teamsmiley 

![]({{ site.baseurl }}/assets/sync-7.PNG)

![]({{ site.baseurl }}/assets/sync-8.PNG)

이걸 누르고 복사를 하면 거기에 코드가 있다. 따로 이코드를 적어둔다.

```html
<script src="https://gist.github.com/teamsmiley/5e96db765227cc7ff82def22346b1a42.js"></script>
```

## 다른컴퓨터에서 Setting Sync를 설치하고 alt + shift + d 를 누른다. 

personal token과 Gist id를 넣어줘야 한다.

![]({{ site.baseurl }}/assets/sync-10.png)

![]({{ site.baseurl }}/assets/sync-11.png)

이제 싱크가 완료되서 설치된 플러그인들과 세팅을 sync한다. 


## todo

### macosx와 windows에서 차이점은 없을가?



