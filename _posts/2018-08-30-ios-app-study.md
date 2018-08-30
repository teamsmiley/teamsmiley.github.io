---
layout: post
title: 'ios App 만들기 -1 ' 
author: teamsmiley
date: 2018-08-30
tags: [ios11, swift]
image: /files/covers/blog.jpg
category: {macosx}
---

# 아이폰용 앱을 만들어 보려고 한다.

## 공부하는 법을 적어보자.

목표 : 싱글페이지 앱이며 todolist를 서버에서 받아와서 뿌려주고 스와이프를 통해 완료 체크를 해준다.

미리 준비할것 : oauth인증서버가 있고 (없으면 auth0.com을 사용) api도 만들어져있다고 가정한다. 

추가 기술 : cocoapod , alamofire ,  rxswift, swiftyjson,


1. 기본 프로젝트 만들기 - xcode , new project
1. Controller에 dummy data 만들기 
1. table view를 넣어서 화면에서 data를 가져와서 뿌리기
1. 화면에 검색창 넣기 
1. auto layout 적용
1. 기본 화면에 들어왔을때 인증 체크 - userdefaults 
1. 인증이 안된상태면 화면을 로그인 화면으로 보내자 - userdefaults
1. 인증을 하고 다시 돌아오면 화면에 인증정보를 뿌려주자. - oauth2 필요
1. 인증이 된 상태이므로 restapi를 호출해서 서버에서 데이터를 가져오자. - Alamofire,RxSwift,SwiftyJson
1. 데이터를 가져와서 테이블뷰에 다시로드
1. 검색도 가능하게 하자.(api가 필터를 제공해야한다.)


추가목표 

1. todo의 자세한 내용이 나오는 두번째 화면을 만든다. 
1. todo에서 항목을 선택하면 두번째 화면이 나오게 한다.
1. navigation을 붙여서 뒤로 갈수 잇게 한다.
1. 탭을 추가하여 추가 옵션들을 보여줄수 있다.
1. Silent Refresh (oauth2)

이렇게 같이 공부해보실분 연락주세요 teamsmiley@gmail.com

