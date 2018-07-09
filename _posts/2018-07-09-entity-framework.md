---
layout: post
title: 'Entity Framework Core 2.1 - Tip' 
author: teamsmiley 
date: 2018-07-09
tags: [ef]
image: /files/covers/blog.jpg
category: {program}
---

# Entity Framework Core 2.1  - Tip

2.1부터 lazy loading이 지원된다고 해서 공부  급하게 공부하고 정리해서 틀릴수 있으니 꼭 다시 확인하기 바람. 

##  용어 설명 
* 그래프 : 연관관계가 있는 데이터들을 그래프라고하는것 같음.  team 과 member에서 team A를 선택하면 관련된 팀원들이 쭉 붙는것 이것이 바로 그래프라고 하는듯. 영어강의를 듣다보면 자주 나와서..
* N + 1 문제 : 1번 쿼리를 실행후 그 결과값으로 온 N 개의 데이터를 가져오기 위해 N번의 쿼리를 실행하는것을 말함. 

## ef에서 로딩의 종류 

1. Eager Loading (즉시 로딩) 
1. Lazy Loading (지연 로딩) 

크게 이렇게 두개로 나누어 진다. 

## Lazy Loading 

객체 초기화시 관련 모든 데이터를 가져오지 않고 필요시에 가져온다.

team A를 가져온후 member는 필요시점에 쿼리를 또 실행하여 값을 가져온다.

장점 : 필요없는데이터를 처음부터 안가져오니 메모리도 적게 필요하고 좋다.
문제점  : N+1 문제가 발생한다. (쿼리를 많이 날려보내니 늦어질수도 있음.)

사용법 : n개의 값을 가져온후 루프를 돌면서 쿼리를 다시 실행.


  
```cs
using (var context = new SchoolEntities())
{
  foreach (var department in context.Departments)
  {
    foreach (var course in department.Courses)
    {
      Console.WriteLine("{0}: {1}", department.Name, course.Title);
    }
  }
}
```

## Eager Loading

orm에서 객체를 초기화하는 순간에 관련 데이터들을 다 가져오는 것을 말한다.(간단하게 말하면 조인을 이용해서 한번에 쿼리를 해서 가져온다.) 

예를 들면 Team Table과 Member Table이 있으면 이 둘을 조인해서 결과를 바로 보여준다. 

장점 : 쿼리 1번에 모든 값을 가져올수 있으므로 빠르다.  

단점 : 디비의 량이 많을때는 조인시 부담을 가져올수 있다. 

사용법 : Include 를 사용한다.

```cs
using (var context = new SchoolEntities())
{
  foreach (var department in context.Departments.Include("Courses"))
  {
    foreach (var course in department.Courses)
    {
      Console.WriteLine("{0}: {1}", department.Name, course.Title);
    }
  }
}
```

참고 <http://blogs.microsoft.co.il/gilf/2010/08/18/select-n1-problem-how-to-decrease-your-orm-performance/>


## 해결 방법 

기본적으로 Lazy Loading으로 사용하고 필요한경우에는 Eager Loading을 사용하면될듯.

## core 2.1에서 lazy loading 사용법 

<https://docs.microsoft.com/en-us/ef/core/querying/related-data#lazy-loading>

1. 프록시 사용

* Microsoft.EntityFrameworkCore.Proxies package 설치 
* 디비 설정 
  ```cs
  .AddDbContext<BloggingContext>(
    b => b.UseLazyLoadingProxies()
        .UseSqlServer(myConnectionString));

  ```
* virtual 추가
```
  public class Blog
  {
      public int Id { get; set; }
      public string Name { get; set; }

      public virtual ICollection<Post> Posts { get; set; }
  }

  public class Post
  {
      public int Id { get; set; }
      public string Title { get; set; }
      public string Content { get; set; }

      public virtual Blog Blog { get; set; }
  }
```

2. Lazy-loading without proxies
* Microsoft.EntityFrameworkCore.Abstractions package 설치 
* 나머지는 링크문서 참고 

두 경우 다 Startup.cs를 변경해주자. 
```cs
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddMvc()
        .AddJsonOptions(
            options => options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
        );

    ...
}
```

## TODO

* 가급적 조인을 사용하지 않으려고 하고 있는데 그 이유는 디비를 확장할 경우를 대비해서다. 디비가 확장되면 조인을 못하기 때문이다. 조인을 안하려면 n+1문제를 어떻게 풀지?

* 





