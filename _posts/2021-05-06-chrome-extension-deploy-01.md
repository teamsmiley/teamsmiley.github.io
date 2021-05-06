---
layout: post
title: 'chrome extension deploy'
author: teamsmiley
date: 2021-05-06
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# chrome extension deploy

## 개발자 등록

Chrome 웹 스토어 개발자로 등록 해야한다.

<https://chrome.google.com/webstore/devconsole/register?hl=ko>

)
![]({{ site_baseurl }}/assets/2021-05-06-08-25-34.png)

결제 (5불)를 하면 개발자 대시보드로 이동을 할수 있다. 20개까지 업로드가 가능하다고 한다.

![]({{ site_baseurl }}/assets/2021-05-06-08-28-50.png)

쉽네 하고 로그인한 순간 뭔가 이상하다.

![]({{ site_baseurl }}/assets/2021-05-06-08-32-06.png)

chrome 앱을 지원중단한다고?

검색해보니 크롬앱의 종료가 22년 으로 연기 됫다고 한다.

<https://www.itworld.co.kr/t/54652/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80/160765>

기사 마지막에

> 구글은 확장 프로그램에는 서비스 종료 기간이 적용되지 않는다고 다시 한번 강조했다. 크롬 플랫폼 부서의 기술 이사 앤서니 라포지는 8월 10일 블로그를 통해 이번 변경 사항은 크롬 확장 프로그램 지원에 영향을 미치지 않으며, 구글은 기존 모든 플랫폼에서 계속 크롬 확장 프로그램을 지원하고 투자를 아끼지 않을 것이라고 밝혔다.

크롬 익스텐션과 크롬 앱은 다른것같다. 일단 기사를 믿고 진행

소스코드를 압축해서 zip으로 만들자.

![]({{ site_baseurl }}/assets/2021-05-06-08-38-09.png)

새항목을 누르고 zip파일을 업로드 하면 된다.

![]({{ site_baseurl }}/assets/2021-05-06-08-41-58.png)

![]({{ site_baseurl }}/assets/2021-05-06-08-42-21.png)

대충 넣고 임시저장

제출할수 없는 이유라는것이 잇어서 클릭

![]({{ site_baseurl }}/assets/2021-05-06-09-06-38.png)

간단하게 블로그할려고햇는데 뭐가 자꾸 복잡해짐.

일단 계정에 가서 이메일 추가

개인정보 보호 관행도 추가

이것저것 다 추가해주니 제출할수 잇게 됨.

![]({{ site_baseurl }}/assets/2021-05-06-09-17-15.png)

![]({{ site_baseurl }}/assets/2021-05-06-09-17-29.png)

![]({{ site_baseurl }}/assets/2021-05-06-09-18-10.png)

![]({{ site_baseurl }}/assets/2021-05-06-09-18-28.png)

추가한 내용

![]({{ site_baseurl }}/assets/2021-05-06-09-19-49.png)

![]({{ site_baseurl }}/assets/2021-05-06-09-20-09.png)

제출되는거 보고 나머지는 진행하자.

//todo

크롬 앱 vs 크롬 익스텐션

https://zuminternet.github.io/Zum-Chrome-Extension/ review

액티브 탭 권한과 https http권한도 없어도 되는데 괞히 추가해서 복잡해졌음. 정리 해야함 - 권한 수정해서 업로드 하려고하는데 안됨..검토끝날때까지 안될듯

