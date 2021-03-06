---
layout: post
title: 'Code Style'
author: teamsmiley
date: 2017-02-10'
tags: [code]
image: /files/covers/code.jpg
category: { program }
---

# Code Style (코드 규약) - ver. 0.1

## 최우선 목적

1. 유지보수가 극단적으로 편한 프로그램을 만든다.
2. 개발 기간이 극단적으로 줄어들수 있는 스택을 구성한다.
3. 자동화에 의한 테스트를 이용해서 버그 발생을 방지한다.
4. 중복된 코드가 극단적으로 없어야 한다.
5. architecture should be independent of frameworks

## 프로젝트 기본 셋업

- 프로젝트 폴더에는 docs라는 폴더를 기본으로 만든다.
- 프로젝트와 관련된 문서파일들이 정리되야한다.
- 하위폴더로 개발일지라는 폴더를 하나 만든다.
- 마크다운 문서로 개발일지를 매번 작성한다. (개발일지-1.md)
  - 개발자가 여러명이면 각자 폴더를 만든다.
  - image를 넣어야 하므로 assets/images/폴더도 만든다.
- 개발일지를 쓰거나 프로그램시 궁금증이 생기거나 todo가 발생하면 todos-questions.md 라는 파일을 만들어서 추가해둔다. 완료시 삭제하고 궁금증의 경우 결과를 블로그로 옮겨서 전체 팀원이 참고할수 있게 하고 이문서에서는 지운다.

## 네이밍 규칙 (https://msdn.microsoft.com/en-us/library/ms229043.aspx) 을 정확히 지킨다.

1. 파스칼 : 단어의 첫글자는 대문자.
2. 카멜 : 첫단어의 첫글자는 소문자. 나머지 단어의 첫글자는 대문자.

- 클래스 : 파스칼 : Book
- 멤버변수 : 파스칼 : Receive
- 함수 : 파스칼 : Recieve(int i)
- 함수 - 파라미터 : 카멜 : int players, void SendMessage(string userName)
- 임시 변수 - 카멜
- private변수 - \_(underscre)+ camel case
- 인터페이스 : I 접두어 붙이기
- 약어 사용 금지
- 한글자 사용 금지
- Bool 변수 함수 속성에는 - Is , Has , Can , Should 붙이기 (임시면서면 첫글자 소문자.)
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
- 모든 객체에 tostring을 override하자.
- 매직넘버를 사용하지 않기
- 중복을 사용하지 않기
- 모든 api는 ( 컨트롤러는) return값에 http상태 코드를 보내고 conent에 data를 실어 보내야한다.
  - Put은 StatusCode(HttpStatusCode.NoContent)를 결과로 던지자.
  - Post는 꼭 새로 생성된 아이디를 받아야한다...
- 모든 컨트롤러는 테스트가 작성이 되어야한다.
- 모든 컨트롤러는 entity model을 직접 사용하지 말고 view model 을 만들어서 사용하자.
  - 심지어는 같은 클래스일 경우에도 복사를 하더라도 이 패턴을 지키자.
- 컨트롤러에서 절대 username을 받지 말자.
- 프론트앤드에서는 절때 username 넘기지 말기.
- api단이 노출하는 것과 프론트앤드에서 넘기는것이 일치하게 (과하게 노출하는것 금지)
- 비지니스 로직은 서비스 레이어에서.
- 각 클래스는 각각의 파일에 있어야한다.
- 클래스에서 함수 순서는 abc순으로 한다. GET ADD IS 등은 무시한다.
- 서비스 혹은 매니져 레이어는 기존에는 있엇으나 리팩토링하면서 같이 하면 너무 복잡하여 일단은 없는걸로 간주 컨트롤러에 로직을 넣기고 하겠음. 리팩토링이 완료되면 서비스 레이어를 나눌것임.
- 컨트롤러는 무조건 간단해야하며 로직이 들어가서는 안됨(위에꺼가 우선이므로 일단 보류)
- 컨트롤러는 어떤 로직도 포함하면 안된다
- 결합도는 낮추고 응집도는 높여라.
- separation of concerns; SoC; 관심사의 분리 꼭 지키자. <http://zetawiki.com/wiki/관심의_분리_SoC>
- 화면에 완성된 데이터가 넘어가야한다. (예를 들면 인보이스에 전체 금액이 넘어가야지 금액 , 할인률이 넘어가서 화면에서 계산을 해서는 안된다.) edit화면의 경우는 예외로 한다.

## 패턴

- UnitOfWork 패턴과 리파지토리 패턴은 알아두자.
- 이벤트 패턴은 꼭 이해하고 사용하자.
  - https://teamsmiley.github.io/2016/12/27/c-3/

## unit of work

- 리파지토리 밖에서 iqueryable을 이용한 체인을 걸어서 쿼리하지 마라.
- 리파지토리는 언제나 IEnumerable을 리턴한다.
- 절때 IQueryable을 리턴해서는 안된다. - 리파지토리 밖에서 쿼리를 만들어버릴 수 있으므로 원천 봉쇄해야한다.
- 리파지토리에는 어떤 비지니스 로직도 들어가서는 안된다.
- DBContext는 private field on the repository , do not expose outside of repository
- repository에서 dbcontext를 받는다. 특정 컨텍스트가 아니라 제너릭임 - 어플리케이션 부분이 아님
- 디비 쿼리 로직에 대한 모든 내용은 리파지토리에 작성되야한다.
- 리파지토리는 디비와의 연관만 담당한다.
- 언제든 리파지토리만 바꿈으로써 orm이나 디비를 바꿀수 있다고 생각해야한다.
- 리파지토리에는 save/ update 가 없다.
- Update는 unit of work에서 처리 - complete(); 호출 처리
- Add는 memory에 추가후 complete(); 호출
- 테이블당 하나의 리파지토리 클래스.
- repository도 어떤 비지니스 로직을 포함하지 말자.

## Paging

- 페이징 관련 용어 통일
- page : 몇번 페이지는 몇장인가
- pagenumber: 몇페이지인가 ⇒ 전체가 몇페이지인지 보여줄필요가 있을까?
- size : 한페이지에 몇개씩 뿌리는가

## Automapper

- 오토매퍼는 데이터의 매핑만 한데 어떤 로직도 들어가면 안됨.
- 오토매퍼는 컨트롤러에서만 사용하자 (서비스레이어에서도 사용이가능 그런데 리파지토리에서는 사용하지 말자.)
- 테스트에서는 \_unitofwork 사용가능한곳
- 컨트롤러 생성자에 넘겨줄때
- 실제디비에서 값을 가져와서 비교하는 부분( fromDB) 에만 사용하자.
- 하나의 함수는 하나의 기능을 해야한다.
- Api가 리턴값을 보낼때는 json으로 해야한다. 몇개만 list로 보내버리면 안됨..무조건 json으로 감싸아서 보내야함..

## ViewModel

- 도메인에서는 뷰모델을 사용하지 말자. 모든 viewmodel은 api프로젝트에 있어야한다.(?)
- Viewmodel을 잘 정해야한다.
- 중복된 viewmodel이 없도록 해야한다..
- 상속이 가능하면 상속을 받아서 처리도 해보자.
- List를 받을때는 꼭 list라는 클래스 이름을 써주자.
- Viewmodel이 너무 크지 않게 해야한다. 꼭 필요한거만 가야한다. 그런데 viewmodel을 너무 많이 만들기 어려우므로 한두개정도 필드는 합쳐서 하나로 만들수도 있다.
- 리파지토리에 페이징을 구현하기 위해서는 전체 개수가 필요하다.

```cs
public IEnumerable<Invoice> GetUserInvoices(SearchUserInvoiceViewModel vm)
        {
            var oldestDate = DateTime.Now.AddMonths(-24);

            return AAAContext.Invoices
               .Where(i => i.UserName == vm.UserName || i.ParentUserName == vm.UserName)
               .Where(i => i.BillingStatus != "UnInvoice")
               .Where(i => vm.Product == "ALL" || i.Product == vm.Product)
               .Where(i => i.BillingDate > oldestDate)
               .OrderByDescending(i => i.Id)
               ;
        }
```

이렇게 원본을 가져오는걸 먼저 만들고(페이징이 적용되기전 데이터)
이걸 이용하는 다음 두개 함수를 만든다.

```cs
public int GetUserInvoicesTotalCount(SearchUserInvoiceViewModel vm)
        {
            return GetUserInvoices(vm).Count();
        }
```

전체 갯수 리턴하는 함수 tolist를 쓰는걸 주의

다음건 전체를 가져와서 페이징을 한 데이터를 리턴한다.

```cs
public IEnumerable<Invoice> GetUserInvoicesWithPaging(SearchUserInvoiceViewModel vm)
        {
            var invoices = GetUserInvoices(vm);

            var result = invoices.Skip((vm.Page - 1) * vm.Size).Take(vm.Size).ToList();

            return result;
        }
```

이렇게 3세트가 필요한듯.
IEnumerable<Invoice> GetInvoices(SearchAdminInvoiceViewModel vm);
IEnumerable<Invoice> GetInvoicesWithPaging(SearchAdminInvoiceViewModel vm);
int GetInvoicesTotalCount(string status, SearchAdminInvoiceViewModel vm)

## Service

- service layer에서는 도메인 오브젝트를 넣고 빼야 한다. Job클래스를 넘겨야지 jobid를 넘겨서 또 가져오지 말자.

## 고민거리

- 프론트앤드에서 newjob 유저에서 사용하나 어드민에서도 사용한다.=> 어드민에서 user쪽 api를 사용하는것은 괜찮다.(두번 중복할 필요가 없는것 같음.)
- 프론트앤드에서의 service 여러 곳에서 사용하기 위해서 만들어둔것.
- 프론트앤드에서 api url이 검색으로 확인이 됬으면 좋겟음.
- 코드가 명확했으면 좋겟음.
- controller에서 다음코드가 실행되는데 이건 리파지토리에서만 디비관련 코드가 있다라는 규칙을 어긴건가? 이게 편한데...

```cs
public IActionResult CreateCourse([FromBody] CourseCreateResource resource){
    ...
    _unitOfWork.Courses.Add(entity);
    _unitOfWork.Complete();
    ...
}
```
