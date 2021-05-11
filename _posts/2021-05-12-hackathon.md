---
layout: post
title: 'Hackathon 그후'
author: teamsmiley
date: 2021-05-12
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# Hackathon 그후

Hackathon에 간단히 advisor로 참여한 프로젝트가 있다. 시간이 너무 촉박하여 프로젝트를 마무리하는데까지만 집중을 햇었었다. 그 후 추가 진행된 내용을 정리해보려고 한다.

## 프로젝트 설명

간단하게 크롬에서 새창을 띄우면 스트레칭 할수 잇는 화면을 보여주는 것이엿다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-13-17.png)

로컬에서 빌드후 aftifact를 압축해서 사용햇다.

## chrome web store에 등록

크롬웹스토어에 등록하는 부분은 기존 글에 있으니 참고하시고 1차 버전은 아무것도 없는 기본 템플릿여서 검수에서 통과 되지 못햇다. 프로젝트 완료후 완료된 버전으로 검수를 넣엇고 2틀후 검수통과를 받았다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-15-28.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-15-46.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-16-17.png)

## staic website launch

s3에도 같은 내용의 사이트를 업로드 하여 웹으로 접속도 가능하게 해두었다.

빌드시 자동으로 s3에 업로드 되게 해두었다 관련 내용은

<https://teamsmiley.github.io/2021/05/11/github-action/> 여기에 정리되있다.

## 아이콘 추가

배포된 내용을 설치해보니 아이콘이 안보이는것을 확인할수 있엇다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-19-23.png)

작업하여 manifest에 icon만 추가해주면 처리가 되는것을 확인햇다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-20-00.png)

## 크롬 웹 스토어에 자동 업로드

소스코드 업데이트는 끝낫고 빌드시 자동으로 크롬 웹 스토어에 업로드를 해야한다.

깃헙 액션에서 빌드후 자동으로 업로드가 되게 해두었다.

처리 관련 내용은 <https://teamsmiley.github.io/2021/05/07/chrome-extension-deploy-02/>

## 버전 업데이트 실패

새버전을 업로드 해보니 퍼블리시가 안된다. 왜냐면 version이 manifest파일에 있는데 여전히 0.0.1 이여서 .

새버전은 버전이 업데이트 되어야하나보다. 0.0.2로 바꾸고 마스터에 커밋/푸시 하면 잘 올라간다.

## 다시 검수

Chrome 웹 스토어 개발자 대시보드<> 에 가서 새버전 검수를 다시 요청한다. 잘 되나 보자.

<!-- ## 구글 애널리스틱

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-27-39.png)

구글 애널리스틱으로 앱스토어에서 데이터가 나오나 보다.

추가해보자.

구글 애널리스틱 사이트에서 계정을 추가한다. <https://analytics.google.com/>

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-29-15.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-29-56.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-30-26.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-30-53.png)

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-31-34.png)

생성 완료후 다음 처럼 보인다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-33-12.png)

대충 된거같다. 지금은 스토어에서의 visit수만 나오겟지만 나중에 app에 추가하여 트래픽을 해봐도 좋을거같다. 그때는 이 속성을 추가하면 될듯

계정 아이디를 확인한후 그걸 크롬 웹 스토어에 넣어주면 되보인다.

![]({{ site_baseurl }}/assets/2021-05-12-hackathon/2021-05-11-08-35-06.png)

애널리스틱 번호 : 196746140
속성 번호 : 271985253 -->

## 결론

바쁜 해커톤 과정에는 프로덕트에 집중할수 밖에 없지만 지나고 나서는 ci/cd 그리고 데이터를 확인할수 있는 구글 애널리스틱 등도 다 작업을 해두고 자동화로 모든걸 볼수 있어야 적은 노력으로 프로젝트의 진행을 더 많이 할수 있을듯 하다.

## todo

- version 자동 업로드
- 구글 애널리스틱
