---
layout: post
title: 'AutoMapper -2 심화'
author: teamsmiley
date: 2016-12-23
tags: [AutoMapper]
image: /files/covers/blog.jpg
category: { c# }
---

# AutoMapper - 2

## 왜 사용하는가? - 도메인 객체와 viewmodel사이를 매핑해주기 위해서 사용한다. 중요한 것은 전체 어플리케이션에서 한번만 매핑관련 정보를 세팅하고 그걸 계속 사용한다는 것이다.

## 기본 사용법

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

### 중복되지 않은 프로퍼티에 대한 정리

PreBalance는 뷰모델에 넘기지 않는 프로퍼티이다. 오토 매퍼는 이 프로퍼티를 처리하는 방법을 몰라서 에러를 낸다.
다음처럼 정의해줘야 한다.

```cs
cfg.CreateMap<Invoice, InvoiceDetailViewModel>()
        .ForMember(dest => dest.PreBalance, opt => opt.Ignore())
        ;
```

### PO가 널일 경우는 - 로 표시되게 한다.

```cs
cfg.CreateMap<Invoice, InvoiceDetailViewModel>()
         .ForMember(dest => dest.Po, opt => opt.NullSubstitute("-"))
         ;
```

### 상속받은 객체를 매핑하는 방법

```cs
class App
{
	public string Name;
}

class Version
{
	public string Name;
	public  App App { get; set; }
}

class Plugin
{
	public string Name
	public Version Version;
}
```

추가 설명을 하면 plugin 이 version을 가지고 있고 versoin이 app을 가지고 있는 구조이다.

이와 같을때 뷰모델

```cs
public class ViewModel
{
	public string P_Name { get; set; }
	public string A_Name { get; set; }
	public string V_Name { get; set; }
}
```

```cs
cfg.CreateMap<Plugin, ViewModel>()
	.ForMember(dest => dest.A_Name, opt => opt.MapFrom(src => src.Version.App.Name))
	.ForMember(dest => dest.V_Name, opt => opt.MapFrom(src => src.Version.Name))
	.ForMember(dest => dest.P_Name , opt => opt.MapFrom(src => src.Name)
	;
```

혹시 잘못된부분이 있으면 teamsmiley@gmail.com으로 이메일 부탁 드립니다.

관련 글

AutoMapper - 1 <https://teamsmiley.github.io/2016/12/22/automapper-1>

AutoMapper - 2 <https://teamsmiley.github.io/2016/12/23/automapper-2>

AutoMapper - 3 <https://teamsmiley.github.io/2016/12/23/automapper-3>

AutoMapper - 4 <https://teamsmiley.github.io/2016/12/23/automapper-4>

AutoMapper - 5 <https://teamsmiley.github.io/2016/12/26/automapper-5>

## 동영상 강의

http://channel9.msdn.com/posts/ASPNET-MVC-With-Community-Tools-Part-10-AutoMapper

http://www.dnrtv.com/?showNum=155

## 참고 사이트

https://github.com/AutoMapper/AutoMapper/wiki

http://funnygangstar.tistory.com/entry/AutoMapper%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-%EA%B0%9D%EC%B2%B4%EA%B0%84-%EB%A7%B5%ED%95%91
