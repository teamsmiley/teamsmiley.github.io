---
layout: post
title: 'iis rewrite' 
author: teamsmiley
date: 2018-07-30
tags: [iis,rewrite]
image: /files/covers/blog.jpg
category: {iis}
---

# iis rewrite 

설치 부분은 빼고 적어나가겠습니다. 

참고 : <https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference#Accessing_URL_Parts_from_a_Rewrite_Rule>

위 참고를 보면 모두 알수 있다. 

url이 다음과 같다고 하자 

http://www.google.com/search?q=IIS+url+rewrite

이것은 다음과 같이 자를수 있다. 

http(s)://<host>:<port>/<path>?<querystring>

Server Variable로 각각의 값들을 가져올수 있다. 

| 내용   |      값      |  메모 |
|:----------|:-------------|:------|
|SERVER_PORT_SECURE or HTTPS = on/off| http(s)| 프로토콜|
|HTTP_HOST | www.google.com | 호스트명 /가 없다|
|SERVER_PORT|    |Default is 80|
|URL(아래 그림에서 A 부분에서 사용) |  search/term   | /가 없는것을 주의한다. ?표 앞까지 나중에 룰을 만들때 /로 시작하게 하면 매칭이 안된다.|
|PATH_INFO | /search | /가 있다|
|QUERY_STRING | q=IIS+url+rewrite | 전체 쿼리 스트링|
|REQUEST_URI| /search?q=IIS+url+rewrite | host와 포트를 제외한 모든것 | 


슬래시가 어디까지 붙는지가 중요하다. 잘못하면 룰이 매칭이 안된다. 

iis-rewrite-01.PNG 에서 A부분에 사용 

이름: 룰의 이름을 정의한다. 

pattern : regex에서 매치할 이름을 정의한다. 예를들면 .* 는 모든 url을 가르킨다. 

이렇게 match url을 만들고 Action을 설정한다. (정규식을 알아야 한다. ^ 는 글의 시작 $는 끝 . 은 아무문자 * 은 0-여러번 반복. 결과적으로 .* 는 모든 문자열을 말한다.)

Action은  redirect , rewrite , Abort Reqest 등이 많이 사용된다. 

rewrite를 할때는 url을 써주면 된다. 

마지막으로 stop processing of subsequent rule을 체크해주자. 이건 이 패턴에 매치되면 더이상 다른 패턴을 찾지 않는것으로 보이나 정확하지는 않다. 

아래처럼 xml을 만들어서 web.config 에 넣어도 동작을 한다. 

```xml
<rule name="Lang-EN" stopProcessing="true">
  <match url="^en[\/].*?" />
  <action type="Rewrite" url="dist/en/server.js" />
</rule>
```
url이  en으로 시작하는것은 ('/en'이 아니다.)  /dist/en/server.js로 보내라는 뜻이다. 


다음을 보자. 모든 url에 대해 ('.*') https://로 보내라 하는 것인데.

```xml
<rule name="Redirect to HTTPS" stopProcessing="true">
  <match url="(.*)" />
  <conditions>
    <add input="{HTTPS}" pattern="^OFF$" />
  </conditions>
  <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" appendQueryString="true" />
</rule>
```
새로운것이 나왔다 condition (조건)  위에서 한번 조건을 주었지만 또 하위 조건을 주는 것이다. 보면 https 라는 서버값이 off 면 아래 action을 하라는 이야기다. 

또 다른 예를 보자. 

```xml
<rule name="Convert to lower case" stopProcessing="true">
  <match url=".*[A-Z].*" ignoreCase="false" />
  <action type="Redirect" url="{ToLower:{R:0}}" redirectType="Permanent" />
</rule>
```

url에 대문자가 있으면(대소문자 무시하지 말고) ToLower 소문자로 바궈서 redirect해라  이것이다. 

R:0은 아직 뭔지 모르겟다. 다만 실서버에서 gui를 이용하면 패턴을 테스트하는 버튼이 잇는데 해보면 저 값을 알려준다.

c:0도 있는듯 싶다. 

하나 더 해보자.

```xml
<rule name="Asset Redirect" stopProcessing="true">
  <match url="^assets\/.*" />
  <action type="Rewrite" url="dist/en/server.js" appendQueryString="true" />
</rule>
```

url이 assets/로 시작하는것은 모두 /dist/en으로 보내라 이것이고 쿼리스트링은 붙여서 보내라 이것이다. 

다음은  aaa.com 처럼 url이 아무것도 없는 경우 /en/home으로 보내라 라는것이다. 

```xml
<rule name="Root Hit Redirect" stopProcessing="true">
    <match url="^$" />
    <action type="Redirect" url="/en/home" />
</rule>
```

## 궁금증 

* {R:0} : 뭐지?
* {C:0} : 뭐지?
* 룰의 우선순위는 어떻게 되나? => 내생각에는 위쪽부터 내려오면서 매치가 되는것에 stopProcessing="true"가 있으면 더이상 진행하지 않는듯 하다 그래서 아래쪽 룰이 적용이 안되는듯 하다. 순서도 주의해서 사용해야 할듯 싶다. 

끝내 위 궁금증은 못풀고 매뉴얼을 작성을 마무리 하게 되었다.

누가 아시는분은 댓글 부탁드립니다. 

## 참고 

* <https://www.youtube.com/watch?v=hkEFPzixiVE>

* <http://www.egocube.pe.kr/translation/content/url-rewrite/200807020001>
