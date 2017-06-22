--- 
layout: post 
title: "google tag manager angularjs" 
date: 2017-06-21 00:00  
author: teamsmiley 
tags: [entity]
image: /files/covers/blog.jpg
category: {entity}
---

# Google Tag Manager 를 Angularjs 로 사용해보기 

이걸 하게 되면 앞 두 포스트에서 복잡하게 하던걸 한방에 해결한다. 

## Google tag manager에 가입한다. 
https://tagmanager.google.com

## 새계정 만들기 

가입하고 관리자에 가서 새계정을 만든다. 
## 컨테이너 만들기 

이제 컨테이너를 하나 만든다. 

## tag trigger 추가하기

* 만들고 나면 작업공간 메뉴>>  tag를 추가한다. 
* 태그 구성에서 유니버설 애널리틱스를 선택하고 나면 추적 유형은 페이지뷰 구글 애널리스틱설정은 설정 변수 선택이라는 드롭박스가 나오는데 새 변수 선택하면된다.
* 기존에 애널리틱스에서 만들어둔 ua로 시작하느 번호를 넣어주면 된다. 
* 이제 트리거를 만들어야한다. 모든 페이지를 선택하고 완료.
## 미리보기로 확인하기 

이제 상단에 메뉴를 보면 미리보기가 있다. 
누르고 나서 탭을 하나 띄운후 실제 사이트를 띄우면 디버깅 창이 뜨면서 결과값을 보여준다.

## 게시 
이제 게시를 하면된다. 

## 결과 확인 
구글 애널리틱스 화면에서 실시간을 보면 페이지를 리프레시 하면 화면에 보인다.

## 참고
* 잘되는 지 확인을 하려면 크롬 앱 ExtensionTag Assistant을 설치하면 편하다. 
<https://chrome.google.com/webstore/detail/tag-assistant-by-google/kejbdjndbnbjgmefkgdddjlbokphdefk>
* 기존 문서에서 app.js에서 추가하던 다음 부분은 삭제해도 된다.
```
$window.ga('send', 'pageview', { page: $location.url() });
```
* 기존 문서에서 index.html 에 추가하던 부분은 삭제한다. 
```html
<!-- google analytics -->
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-101388139-1', 'auto');
  ga('send', 'pageview');
</script>
```
* 위 두개를 처리하지 않으면 두번씩 로그가 잡히기도 한다. 










