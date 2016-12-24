---
layout: post
title: 'AutoMapper-3-customvalueresolver' 
author: teamsmiley 
date: 2016-12-23
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: {c#}
---

# AutoMapper - 3 - customvalueresolver 

## 왜 사용하는가? - 도메인 객체와 viewmodel사이를 매핑해주기 위해서 사용한다.  중요한 것은 전체 어플리케이션에서 한번만 매핑관련 정보를 세팅하고 그걸 계속 사용한다는 것이다. 

```cs
public class DBEntity
{
    public int Value1 { get; set; }
    public int Value2 { get; set; }
}

public class ViewModel
{
    public int Total { get; set; }
}
```

### 전체 프로그램에서 DBEntity를  ViewModel객체로 매핑시  total에 value1과 value2를 합쳐서 보여주고싶다. 

DBEntity => ViewModel로 매핑설정 

```cs
cfg.CreateMap<DBEntity, ViewModel>()
         .ForMember(dest => dest.Total, opt => opt.ResolveUsing<ViewModelTotalResolver>());
```

ViewModelTotalResolver 객체를 만들자. 

```cs
public class ViewModelTotalResolver : IValueResolver<DBEntity, ViewModel, int>
{
    public int Resolve(DBEntity dbEntity, ViewModel viewModel, int destMember, ResolutionContext context)
    {
        return dbEntity.Value1 + dbEntity.Value2;
    }
}
```

복잡해 보이지만 간단하다. IValueResolver 만 구현하면된다.

value 1과 value2를 더해서 값을 리턴한다. 
그러면 설정에서 그걸 total에 넣는다. 

```cs
cfg.CreateMap<DBEntity, ViewModel>()
         .ForMember(dest => dest.Total, opt => opt.ResolveUsing<ViewModelTotalResolver>());
```

참고 : 리턴타입을 잘 맞춰야한다. 

![]({{site_baseurl}}/assets/automapper_custom_resolver_01.png)


```cs
public class Invoice
{
    public int Id { get; set; }
    public string UserName { get; set; }
    public DateTime? BillingDate { get; set; }
    public string Po { get; set; }
    public string PreBalance { get; set; }
}

public class InvoiceDetailViewModel
{
    public int Id { get; set; }
    public string UserName { get; set; }
    public string BillingDate { get; set; }
    //DateTime이 아니라 string 인것을 확인
    public string Po { get; set; }
}
```


### 전체 어플리케이션에서 Entity 에서 '2017-12-31- 01:01:01' (DateTime) 형태로 넘어온걸 viewmodel로 보낼때는 '2017-12-31' 로 넣고 싶다. 
```cs 
public class BillingDateFormatter : IValueResolver<Invoice, InvoiceDetailViewModel, string>
{
    public string Resolve(Invoice invoice, InvoiceDetailViewModel vm, string destMember, ResolutionContext context)
    {
        return invoice.BillingDate.Value.ToShortDateString();
    }
}

 cfg.CreateMap<Invoice, InvoiceDetailViewModel>()
          .ForMember(dest => dest.BillingDate, opt => opt.ResolveUsing<BillingDateFormatter>())
          ;
```

이렇게 하면 매핑시 항상 날짜를 shortdatestring으로 매핑한다. 

### : 이름앞에 Mr. 를 항상 붙이고 싶다. 

```cs
public class MrResolver : IValueResolver<Invoice, InvoiceDetailViewModel, string>
{
    public string Resolve(Invoice invoice, InvoiceDetailViewModel vm, string destMember, ResolutionContext context)
    {
        return "Mr. "+ invoice.UserName;
    }
}
```

혹시 잘못된부분이 있으면 teamsmiley@gmail.com으로 이메일 부탁 드립니다.

## 동영상 강의

http://channel9.msdn.com/posts/ASPNET-MVC-With-Community-Tools-Part-10-AutoMapper

http://www.dnrtv.com/?showNum=155

## 참고 사이트 

https://github.com/AutoMapper/AutoMapper/wiki

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91


