--- 
layout: post 
title: "google analytics angularjs" 
date: 2017-06-21 00:00  
author: teamsmiley 
tags: [entity]
image: /files/covers/blog.jpg
category: {entity}
---

# Google Analytics 를 Angularjs 로 사용해보기 

## Google Analytic에 가입한다. 
https://analytics.google.com 

## 코드 확인  

* 관리 >> 계정 >> 속성 >> 추적정보 >> 추적 코드 클릭 

```html
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-101388149-1', 'auto');
  ga('send', 'pageview');

</script>
```

이런 비슷한 코드가 있다. 
이걸 복사한다. 

## index.html  수정 
위에 코드를 앵귤러 앱에 index.html파일의 맨 마지막에 추가한다. 

## 문제점 
Angular는 처음한번 코드를 다운받고 내부적으로 화면을 수정하므로 이 코드가 실행이 되지 않는다. 

## 해결책 

* app.js를 수정한다. 
```js
app.run([... function (...) {
  ...
    $rootScope.$on("$viewContentLoaded", function (event) {
        $window.ga('send', 'pageview', { page: $location.url() });
    });
}]);
```

페이지가 바뀔때 마다 스크립트가 실행되서 구글에 정보를 보낸다. 

## 데이터 확인.

위 작업을 끝낸후 디플로이 하면 약 10분 내로 데이터들이 들어오기 시작하면서 데이터가 보이기 시작한다. 

