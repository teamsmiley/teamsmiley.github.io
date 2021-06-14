---
layout: post
title: 'Program Rule'
author: teamsmiley
date: 2021-06-14'
tags: [code]
image: /files/covers/code.jpg
category: { program }
---

# Program Rule - ver. 0.2

## 최우선 목적

1. 유지보수가 편한 프로그램을 만든다.
2. 개발 기간이 줄어들수 있는 스택을 구성한다.
3. 자동화에 의한 테스트를 이용해서 버그 발생을 방지한다.
4. 중복된 코드가 극단적으로 없어야 한다.(중복에 대한 생각은 조금 바뀌었음 상황에 따라서 중복허용)
5. architecture should be independent of frameworks
6. clean architect를 기본으로 한다. (use case를 기준으로)

## 프로젝트 기본 셋업

- 프로젝트 폴더에는 docs라는 폴더를 기본으로 만든다.
- 프로젝트와 관련된 문서파일들이 정리되야한다.

## 네이밍 규칙

1. (https://msdn.microsoft.com/en-us/library/ms229043.aspx)을 정확히 지킨다.
1. 파스칼 : 단어의 첫글자는 대문자.
1. 카멜 : 첫단어의 첫글자는 소문자. 나머지 단어의 첫글자는 대문자.

- 클래스 : 파스칼 : Book
- 멤버변수 : 파스칼 : Receive
- 함수 : 파스칼 : Recieve(int i)
- 함수 - 파라미터 : 카멜 : int players, void SendMessage(string userName)
- 임시 변수 - 카멜
- private변수 - \_(underscre)+ camel case
- 인터페이스 : I 접두어 붙이기
- 약어 사용 금지
- 한글자 사용 금지
- Bool 변수 함수 속성에는 - Is , Has , Can , Should 붙이기 (임시변수는 첫글자 소문자.)
- 클래스 변수 ( 프라이빗이나 프로텍트 ) - \_(언더스코어)를 단어앞에 붙인다.
- 객체의 이름이 암시되어 있으면 메소드 이름에 반복하지 말자.
- N은 객체의 수를 나타내자.
- 복수는 복수형으로 사용하자.
- Id는 파스칼 id는 카멜 (ID는 틀림.)
- 퍼블릭 함수 이름은 항상 대문자로 시작.

## 코드 관련

- 함수는 10줄 이하로 한다.
- 함수에 아규먼트는 3개 이하로 한다.
- 함수는 하나의 역할만 한다.
- out parametersms 절대 쓰지 않는다.
- 절대 절대경로를 사용하지 않는다.
- 모든 객체에 tostring을 override하자.(이건 자동으로 안해주나)
- 매직넘버를 사용하지 않기
- 중복을 사용하지 않기 (가끔 허용)
- 모든 api는 (컨트롤러는) return값에 http상태 코드를 보내고 content에 data를 실어 보내야한다.
  - Put은 StatusCode(HttpStatusCode.NoContent)를 결과로 던지자.
  - Post는 꼭 새로 생성된 아이디를 받아야한다.(location도 포함)
- 모든 컨트롤러는 테스트가 작성이 되어야한다.
- 모든 컨트롤러는 entity model을 직접 사용하지 말고 view model 을 만들어서 사용하자.
  - 심지어는 같은 클래스일 경우에도 복사를 하더라도 이 패턴을 지키자.
- 컨트롤러에서 절대 username을 받지 말자.
- 프론트앤드에서는 절때 username 넘기지 말기.(jwt로 서버에서 해석)
- api단이 노출하는 것과 프론트앤드에서 넘기는것이 일치하게 (과하게 노출하는것 금지,vm이 정확해야함.)
- 비지니스 로직은 서비스 레이어에서.
- 각 클래스는 각각의 파일에 있어야한다.
- 클래스에서 함수 순서는 abc순으로 한다. GET ADD IS 등은 무시한다.
- 컨트롤러는 어떤 로직도 포함하면 안된다
- 결합도는 낮추고 응집도는 높여라.
- separation of concerns; SoC; 관심사의 분리 꼭 지키자. <http://zetawiki.com/wiki/관심의_분리_SoC>
- 화면에 완성된 데이터가 넘어가야한다. (예를 들면 인보이스에 전체 금액이 넘어가야지 금액 , 할인률이 넘어가서 화면에서 계산을 해서는 안된다.) edit화면의 경우는 예외로 한다.
- 디버깅을 위한 출력문을 만들고 싶을때는 Test code를 작성한다.
- 하나의 함수는 하나의 기능을 해야한다.

## Paging 관련 용어 통일

- page : 몇번 페이지는 몇장인가
- pagenumber: 몇페이지인가 ⇒ 전체가 몇페이지인지 보여줄필요가 있을까?
- size : 한페이지에 몇개씩 뿌리는가

## ViewModel

- 도메인에서는 뷰모델을 사용하지 말자. 모든 viewmodel은 api프로젝트에 있어야한다.(?)
- Viewmodel을 잘 정해야한다.
- 중복된 viewmodel이 없도록 해야한다.
- List는 단수 + Collection 클래스 이름을 써주자.
- Viewmodel이 너무 크지 않게 해야한다. 꼭 필요한거만 가야한다. 그런데 viewmodel을 너무 많이 만들기 어려우므로 한두개정도 필드는 합쳐서 하나로 만들수도 있다.
