---
layout: post
title: 'ef core - Spatial Data' 
author: teamsmiley
date: 2020-06-23
tags: [entity framework]
image: /files/covers/blog.jpg
category: {entity framework}
---

# entity framework core로 공간 정보 계산하기

mysql 8.0을 사용중이다. 

EF Core 2.2 부터는 공간 정보(Spatial Data) 를 저장할수 있게 지원한다. sql server와 sqlite , InMemory , PostgreSQL 는 지원을 했었다. 나는 mysql을 사용하는데 얼마전 Pomelo.EntityFrameworkCore.MySql 의 nightly build에서 지원을 한다는 걸 보고 테스트하기로 햇다.

* <https://docs.microsoft.com/en-us/ef/core/modeling/spatial>
* <https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/issues/338>


## nuget package 설치 

### nightly build 

nightly build를 설치해야해서 조금 헤멧지만 다음처럼 해보자.

<https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql#nightly-builds>ㅍㅍ
솔류션 루트에 nuget.config 파일을 만들고 
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="Pomelo" value="https://pkgs.dev.azure.com/pomelo-efcore/Pomelo.EntityFrameworkCore.MySql/_packaging/pomelo-efcore-public/nuget/v3/index.json" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

을 만들어두고 vs를 시작한다.

### 빌드시 Nuget.config가 복사되게
나의 경우에는 ci/cd를 사용함으로 원격 빌드시 이 파일이 꼭 복사가 각 프로젝트별로 되게 만들어줬어야한다.

하나이  솔류션에 프로젝트가 여러개라 다음처럼 처리 dockerfile을 수정해주었다.

```dockerfile
COPY ["Src/Application/Application.csproj", "Application/"]
COPY ["NuGet.config", "Application/"]
COPY ["Src/Domain/Domain.csproj", "Domain/"]
COPY ["NuGet.config", "Domain/"]
COPY ["Src/Infrastructure/Infrastructure.csproj", "Infrastructure/"]
COPY ["NuGet.config", "Infrastructure/"]
COPY ["Src/WebAPI-User/WebAPI-User.csproj", "WebAPI-User/"]
COPY ["NuGet.config", "WebAPI-User/"]

RUN dotnet restore "WebAPI-User/WebAPI-User.csproj"
```

문제없이 빌드된다.

### nuget 설치

nuget package manager를 열고 오른쪽 위에 pomelo를 선택한다.

![]({{ site_baseurl }}/assets/2020-06-23-15-50-47.png)

browse를 누르면 아무거도 안보인다. include prerelease를 눌러주자. 

![]({{ site_baseurl }}/assets/2020-06-23-15-51-49.png)

이제 2개의 패키지가 보일것이다.  이것들은 프로젝트에 설치한다. 

나의 경우에는 다음 프로젝트에 설치해줫음

* Pomelo.EntityFrameworkCore.MySql -> infrastructure
* Pomelo.EntityFrameworkCore.MySql.NetTopologySuite -> domain 

## migration 준비 
migration을 위해서 EFCoreDesignTimeService.cs 클래스를 만든다. infrastructure 프로젝트에 넣어줬다.
```cs
public class EFCoreDesignTimeService : IDesignTimeServices
{
  public void ConfigureDesignTimeServices(IServiceCollection serviceCollection)
      => serviceCollection.AddEntityFrameworkMySqlNetTopologySuite();
}
```

이제 dbcontext설정에 다음처럼 추가해주자.
```cs
services.AddDbContext<ApplicationDbContext>(options =>
{
  options.UseMySql(
    configuration.GetConnectionString("DefaultConnection")
    , x => x.UseNetTopologySuite()
  );
});
```

## entity에 location을 추가한다.

Point 클래스를 사용한다.

```cs
public class Restaurant 
{
  public Guid Id { get; set; }
  public Point Location { get; set; }
}
```

이제 마이그레이션 하고 디비 업데이트를 한다.

```bash
add-migration InitData -c ApplicationDbContext  -o ./Migrations
Update-Database -c ApplicationDbContext
```

## Create 할때 로케이션을 넣어보자.

srid 는 따로 공부하기 바란다. 자세한 내용은 모르고 그냥 이걸로 다들 쓰길래 사용

Coordinate에 위도 경도를 넣어줘서 포인트를 만든후 디비에 넣어준다.

```cs
var entity = new Restaurant();

var geometryFactory = NtsGeometryServices.Instance.CreateGeometryFactory(srid: 4326);//4326 refers to WGS 84, a standard used in GPS and other geographic systems.
entity.Location = geometryFactory.CreatePoint(new Coordinate(-122.121512, 47.6739882));

_context.Restaurants.Add(entity);
```

디비에 잘 추가되는것을 알수 있다.

## 조회를 해보자.

### 현재 위치에서 거리순으로 정렬을 하자.

```cs
var geometryFactory = NtsGeometryServices.Instance.CreateGeometryFactory(srid: 4326);
var currentLocation = geometryFactory.CreatePoint(new Coordinate(-122.121512, 47.6739882));

query = query.OrderBy(c => c.Location.Distance(currentLocation));
```


### 현재 위치에서 조건을 줘보자.

```cs
var geometryFactory = NtsGeometryServices.Instance.CreateGeometryFactory(srid: 4326);
var currentLocation = geometryFactory.CreatePoint(new Coordinate(-122.121512, 47.6739882));
query = query.Where(x => x.Location.IsWithinDistance(currentLocation, 2000)); // 2000 mi? miter? kilomiter? 안쪽만 검색
```

## todo 
단위가 뭔지는 모르겟다. mi, m, km ? 뭐지?





