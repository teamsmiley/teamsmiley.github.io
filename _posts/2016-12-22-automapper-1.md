---
layout: post
title: 'AutoMapper - 1 기본 ' 
author: teamsmiley 
date: 2016-12-22
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {c#}
---

# AutoMapper -1  

## 왜 사용하는가? - 도메인 객체와 viewmodel사이를 매핑해주기 위해서 사용한다.  중요한 것은 전체 어플리케이션에서 한번만 매핑관련 정보를 세팅하고 그걸 계속 사용한다는 것이다. 

## AutoMapper 설치  

nugget package manager 설치

```
Install-Package AutoMapper
```
* mvc프로젝트를  Global.asx에 정의해놓으면 전체 어플리케이션에 적용이 된다.
* webapi 는 Startup.cs 에 정의해두자.


## 기본 사용법 

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
}
```

이제 이 클래스를 사용하는 코드이다. 전체 어플리케이션에 많은곳에 이 코드가 중복된다. 

```cs
    Order order = new Order();
    order.Name = vm.Name;

    order.DeliveryAddress = vm.DeliveryAddress;
    order.DeliveryZipCode = vm.DeliveryZipCode;
    order.EmailAddress = vm.EmailAddress;
    order.OrderDate = DateTime.Now;
    order.DeliveryStatus = 1;
    return View(order);
```
코드가 중복이라 별로 좋지 않다.

수정후 코드는 다음과 같다. 아주 간단해진다. 

```cs 
OrderViewModel OrderCreate = Mapper.Map<Order,OrderViewModel>(Order);
```

객체를 쓰는곳마다 이 매핑해주는 코드가 있다. 특정 코드에는 항상 들어가야할 orderDate가 빠져잇는경우도 있엇다. 그래서 에러가 나기도 함 

AutoMapper를 사용하면 어떻게 매핑을 할지 한곳에  정의해주고 나머지에서는  매핑만 하면  정의된 규칙대로 매핑을 해준다.

다시한번 이야기하지만 전체 어플리케이션에서 단 한번만 매핑관련 정보를 정의하고 나머지는 그냥 사용한다.  

## 추가 예제

```cs 
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

public class CustomerView
{
    public string FullName { get; set; }
    public string Email { get; set; }
    public string HomeAddressCity { get; set; }
}

//설정하자. 
Mapper.Initialize(cfg =>
{
     Mapper.CreateMap<Customer, CustomerView>();
}

//실제 사용시 
CustomerView customerView = Mapper.Map<Customer, CustomerView>(_customer);

```

프로퍼티 이름이 같으면 자동으로 다 맞춰서 매핑된다. 이거 한번만 해두고 나면 전체 어플리케이션에서 매핑만 하면 코드 중복이 줄어든다. 

### FullName 은 어떻게 들어간걸가?

public string GetFullName  에서 Get을 지우고 매핑한다.

### HomeAddressCity 는 어떻게? 

HomeAddressCity  는 Customer.HomeAddress.City 를 매핑..

Customer가 address클래스를 참조하고 있고 거기에 City라는 프로퍼티가 잇으므로 저렇게 자동 매핑이 가능하다..

관련 글 
AutoMapper - 1 <https://teamsmiley.github.io/2016/12/22/automapper-1>
AutoMapper - 2 <https://teamsmiley.github.io/2016/12/23/automapper-2> 
AutoMapper - 3 <https://teamsmiley.github.io/2016/12/23/automapper-3>
AutoMapper - 4 <https://teamsmiley.github.io/2016/12/23/automapper-4>



혹시 잘못된부분이 있으면 teamsmiley@gmail.com으로 이메일 부탁 드립니다.

## 동영상 강의

http://channel9.msdn.com/posts/ASPNET-MVC-With-Community-Tools-Part-10-AutoMapper

http://www.dnrtv.com/?showNum=155

## 참고 사이트 

https://github.com/AutoMapper/AutoMapper/wiki

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91


