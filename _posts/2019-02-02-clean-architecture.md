---
layout: post
title: 'Clean Architecture' 
author: teamsmiley
date: 2019-02-02
tags: [devops]
image: /files/covers/blog.jpg
category: {clean architect}
---

# Clean Architecture

다음 두 강좌면 대충 끝나는듯.

## uncle bob martin 
* <https://www.youtube.com/watch?v=o_TH-Y78tt4&t=1113s>
* <http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html>
### 정리

![]({{site_baseurl}}/assets/clean-architecture-bob.png)

* The Web is a Delivery Mechanism
* Database is Detail 
* A good architecture allows major decisions to be deferred

## Clean Architecture with ASP.NET Core 2.1
* <https://www.youtube.com/watch?v=_lwCVE_XgqI&t=496s>

### 정리

![]({{site_baseurl}}/assets/clean-architecture-dotnet.png)

* Domain Layer 
  * Entities 
  * Value 
  * Objects 
  * Enumerations (꼭 생성자에서 초기화를 하자)
  * Logic 
  * Exceptions

* Application Layer 
  * Interfaces 
  * Models 
  * Logic 
  * Commands / Queries 
  * Validators 
  * Exceptions

* Persistence Layer
  * DbContext 
  * Migrations 
  * Configurations 
  * Seeding 
  * Abstractions

* Infrastructure Layer
  * Implementations, e.g. 
  * API Clients 
  * File System 
  * Email / SMS System 
  * Clock 
  * Anything external

* Presentation Layer
  * SPA – Angular or React 
  * Web API 
  * Razor 
  * Pages 
  * MVC 
  * Web Forms

## 내 노트 
### MediatR 
신의 한수인듯 싶다. <https://github.com/jbogard/MediatR>

밥 아저씨의 request model과 response model을 한방에 해결해준다. 

궁금햇던것은 두번째 비디오에서 automapper를 사용하지 않는것이엿다. MediatR를 만든 사람이 authmapper를 만들었는데 왜 사용하지 않을가? 쓰면 편한데. 라는 궁금증이 아직도 있다.  

샘플 코드에서는 automapper를 사용하지 않고 다음처럼 처리햇다.
```cs
namespace Northwind.Application.Categories.Models
{
    public class ProductPreviewDto
    {
        public int ProductId { get; set; }

        public string ProductName { get; set; }

        public decimal? UnitPrice { get; set; }

        public static Expression<Func<Product, ProductPreviewDto>> Projection
        {
            get
            {
                return p => new ProductPreviewDto
                {
                    ProductId = p.ProductId,
                    ProductName = p.ProductName,
                    UnitPrice = p.UnitPrice
                };
            }
        }
        public static ProductPreviewDto Create(Product product)
        {
          return Projection.Compile().Invoke(product);
        }
    }
}
```

사용은 
```cs
var renderer = ProductPreviewDto.Create(entity);
```

이러면 매핑이 되서 나온다.

Automapper를 왜 안쓴느지는 더 확인이 필요. 아래 링크에서보면 코드가 완전 간단해지는것을 알수 있다. 나도 써야겟다.
<https://github.com/JasonGT/NorthwindTraders/commit/a8356a945387e732a7db622fc7ea4bb51e690339>

### infrastructure에 nofifyService

이것이 어떻게 동작하는지 궁금햇다. 확인해보니 application layer에 interface를 만들고 application은 이 인터페이스로만 코딩을 한다. 

infrastructure에서 이 인터페이스를 구현해서 클래스로 만든다. 

나중에 이걸 di해주면 잘 동작한다.

## 궁금증

* authmapper를 왜 안쓰는가? 
* 인증은 webapi단에서 하는가? 아니면 application layer에서 해야하는것인가?
* 페이징은 webapi단에서 하는가? 아니면 application layer에서 해야하는것인가?
* ddd에서 entity와 vo의 차이는 무엇인가?(immutable)?
* 확인해보니 최신코드는 automapper는 들어가 있는것도 같다.
