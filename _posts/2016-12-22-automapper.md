---
layout: post
title: 'AutoMapper' 
author: teamsmiley 
date: 2016-12-22
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {c#}
---

# AutoMapper  

## 왜 사용하는가? - 도메인 객체와 viewmodel사이를 매핑해주기 위해서 사용한다.  


객체가 두개가 있다. 하나는 디비에 저장하기 위한 엔티티 객체 두번째는  화면에 뿌리기 위한 viewmodel 
```cs
public class Order
{
    public int ID { get; set; }
    public string Name { get; set; }
    public string DeliveryAddress { get; set; }
    public string DeliveryZipCode { get; set; }
    public string EmailAddress { get; set; }
    public DateTime OrderDate { get; set; }

    public double price_1 { get;set; }
    public double price_2 { get;set; }
}


public class OrderViewModel
{
    public int ID { get; set; }
    public string Name { get; set; }
    public string DeliveryAddress { get; set; }
    public string DeliveryZipCode { get; set; }
    public string EmailAddress { get; set; }
   
    public string CardNumber { get; set; }
    public string CardType { get; set; }
    public string CVV { get; set; }
    public string ExpirationMonth { get; set; }
    public string ExpirationYear { get; set; }
    public string BillingAddress { get; set; }
    public string BillingZipCode { get; set; }

    public double total { get;set; }
   
}
```

비슷하지만 조금 다르다…..

화면에서는 viewmodel을 사용한다. 그리고 이걸 백앤드로 보내서 카드 결제를 한후 필요한 정보만 디비에 저장한다. 

그러다 보니 다음 코드가 생긴다.


```cs
    //viewmodel을 order에 넣을때. 반대의 경우도 생긴다...
    Order order = new Order();
    order.Name = vm.Name;
    order.DeliveryAddress = vm.DeliveryAddress;
    order.DeliveryZipCode = vm.DeliveryZipCode;
    order.EmailAddress = vm.EmailAddress;
    order.OrderDate = DateTime.Now;
    order.DeliveryStatus = 1;
    return View(order);
}
```

이렇게 매번 항목들을 넣어줘야한다. 특히 이 객체를 쓰는곳마다 이 매핑해주는 코드가 필요하다. 
automapper를 사용하면 어떻게 매핑을 할지 한곳에  정의해주고 나머지 곳에서 매핑만 하면 그 규칙대로 매핑을 해준다. 
 
## automapper를 사용법 

nugget package manager 설치

Install-Package AutoMapper

이제 테스트 코드를 만들어 보자..
```cs
public void test1()
{
    Mapper.CreateMap<Order, OrderViewModel>()
    Mapper.AssertConfigurationIsValid();
}
```


기본 코드이다 간단하다 order와 OrderViewModel을 매핑해주는것이다..

프로퍼티들의 이름이 같으면 값을 자동으로 넘겨준다. 

AssertConfigurationIsValid 은 자동매핑이 에러나면 에러를 넘겨준다..
 
```cs
public void test2()
{
    Mapper.CreateMap< OrderViewModel ,Order >()
    Mapper.AssertConfigurationIsValid();
}
```
test1 과 test2를 실행해보면 에러가 나눈 부분을 알수가 있다..

두 객체에 항목중  이름이 중복되지 않는 부분에 대한 규칙을 정의 하지 않았기 때문에 automapper가 에러를 낸다. 

Opt.ignore라는 부분은 꼭 필요한거 아니니 없으면 무시해라 라는 부분이다.

항상 한쪽이 크기 때문에 저렇게 된다..

뭐 일단 이렇게 하면되는데..설명을 하려니 어렵다….

 

이 웹사이트에 설명이 잘 나와있기도 하다..일단 웹사이트 소스를 기본으로 설명을 더해보자.

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91

 

예를 들면 다음과 같은 것이 잇따.

public class Customer

    {

        public string FirstName { get; set; }

        public string LastName { get; set; }

        public string Email { get; set; }

        public Address HomeAddress { get; set; }

        public string GetFullName

        {

            get

            {

                return string.Format(“{0} {1}”, FirstName, LastName);

            }

        }

    }

 

    public class Address

    {

        public string Address1 { get; set; }

        public string City { get; set; }

        public string Country { get; set; }

    }

 

 

그런데 실제 view에 넘기고 싶을때는 다음을 넘기고 싶다.

 

public class CustomerView

{

public string FullName { get; set; }

public string Email { get; set; }

public string HomeAddressCity { get; set; }

}

 

그럼 복잡해진다..객체를 다 만들고 매번 넣을때마다 이쪽에서 불러서 저쪽에 넣어줘야한다..

아 코딩하기 싫어…ㅋㅋ…이런부분을 한번의 정의로 딱 정해두고 자동으로 매핑을 하는데 automapper가 사용된다.

실제 코드를 보자..

 

Mapper.CreateMap<Customer, CustomerView>();

 

CustomerView customerView = Mapper.Map<Customer, CustomerView>(_customer);

 

이렇게 해버리면 자동으로 모든값에 들어가버린다…죽인다.

 

일단 궁금한거는 FullName 이 어떻게 값이 들어간걸가?

 

public string GetFullName  에서 Get을 지우고 들어간거임.

 

HomeAddressCity  는 Customer.HomeAddress.City 를 매핑..

Customer가 address클래스를 참조하고 있고 거기에 City라는 프로퍼티가 잇으므로 저렇게 자동 매핑이 가능하다..

 

변수명이 같은건 그냥 매핑이 자동으로 된다.

 

한쪽에는 잇고 한쪽에는 없는경우 작은걸 큰거에 매핑하는건 된다…

그런데 큰걸 작은거에 매핑하려고하면 작은거에는 없는 프로퍼티가 잇어서 매핑이 안된다.

 

맨처음 예에서 보면

 

Order를  OrderViewModel 에 매핑을 하면  문제가 없다…그런데 반대의 경우가 문제이다 creqate form에 는 카드정보를 order에 어디에 매핑해야할지 모르기 때문이다.

 

다음 코드는 내가 실제로 사용해본 테스트이다.

public void AutoMapperTestMethodOrderToCreateOrder()

{

Mapper.CreateMap<Order, OrderViewModel>()

.ForMember(s => s.CardNumber, opt => opt.Ignore())

.ForMember(s => s.CardType, opt => opt.Ignore())

.ForMember(s => s.CardType, opt => opt.Ignore())

.ForMember(s => s.CVV, opt => opt.Ignore())

.ForMember(s => s.ExpirationMonth, opt => opt.Ignore())

.ForMember(s => s.ExpirationYear, opt => opt.Ignore())

.ForMember(s => s.BillingAddress, opt => opt.Ignore())

.ForMember(s => s.BillingZipCode, opt => opt.Ignore());

 

Order Order = new Order

{

Name = “brian”,

DeliveryAddress = “deliveryaddress”,

DeliveryZipCode = “deliveryzipCode”,

OrderDate = DateTime.Now,

DeliveryStatus = DeliveryStatus.NotYetShipped

};

OrderViewModel OrderCreate = Mapper.Map<OrderViewModel>(Order);

Mapper.AssertConfigurationIsValid();

Assert.AreEqual(OrderCreate.BillingAddress, null);

Assert.AreEqual(OrderCreate.DeliveryAddress, “deliveryaddress”);

}

 

 

 

이렇게 카드정보등 order에 없는 변수들은 무시해라 라는걸  정의를 해주면된다..

 

 

공부 시작할때는 문서를  만들면서 해야지 햇는데  일이 있어서 못하다 보니

지금 공부가 많이 된상태에서 기본적인부분 하려고 하니 괭장히 하기기 싫다..

 

 

아무튼 대충 정의를 해서 사용하는것만드로도 코드가 많이 줄어들엇따..실제 코드는 다음처럼 변경됫따.

 

기존 코드는 를 보자 .

 

 

order.Name = request.Name;

order.BillingAddress = request.BillingAddress;

order.BillingZipCode = request.BillingZipCode;

order.DeliveryAddress = request.DeliveryAddress;

order.DeliveryZipCode = request.DeliveryZipCode;

 

이것이 다음처럼 간단해 졋따.

 

OrderViewModel OrderCreate = Mapper.Map<OrderViewModel>(Order);

 

여러곳에 이걸 사용하는 경우 다음번 코드부터는 완전 간단해짐..

 

아무튼 클래스간에 서로 매핑을 한번 정의하고 모든 코드에서 이걸 사용하게 되면 일관성잇고 참 코드 유지보수에 도움이 된다…..

 

mvc프로젝트를 할때는 아까 매핑을 정의하는 부분을 Global.asx인가 거기다 복사해다 넣으면 전체 어플리케이션에 적용이 된다…

 

써보시라 ..참 좋다..코드가 간단해지고 명확해진다..

 

마지막으로 친구 회사에서 일어난 일잇데..디비에서 디비컬럼을 읽어오는데..정규화가 잘못된건지 어쩐지 모르지만 null이 아니여아 하는데 null이 있는 경우가 잇따 이걸 디비에서 읽어와서 프로그램을 할려고 하니까 하나씩 다 체크하고 난리도 아니엿다….이걸 오토 매퍼를 쓰면 간단해진다..

 

널인경우에는 이걸 리턴해라..하는 것이 잇따..

 

샘플 코드를 보자.

 

Mapper.CreateMap<KOAOrder, KOAOrderEditForm>()

                .ForMember(s => s.TrackingUrl, opt => opt.NullSubstitute(“NONE”));

 

 

이걸보면 널이명 “none”를 리턴하게 정의할수잇겟다.. Int면 0을 지정해도 된다…

 

이걸 사용하면 코드가 완전히 간단해 진다….

꼭 사용하기 바란다…

추가 궁금증은 teamsmiley@gmail.com으로 보내면 고민해서 다시 포스팅 할것임..

추가로 동영상 강의 몇 개 추천한다.

http://channel9.msdn.com/posts/ASPNET-MVC-With-Community-Tools-Part-10-AutoMapper

좋다..정말..

 

http://www.dnrtv.com/?showNum=155

이것도 강추…

그리고 아까 이야기한 한글로 된 사이트

 

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91

 

http://smartprogram.tistory.com/5

이것도 참고

이정도면 괞찮을 듯…

매뉴얼 적기 정말 귀찮다….

 

