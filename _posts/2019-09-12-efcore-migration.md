---
layout: post
title: 'ef-core migration ' 
author: teamsmiley
date: 2019-09-12
tags: [devops]
image: /files/covers/blog.jpg
category: {ef core}
---

# ef-core migration 

## add migration 
```
add-migration <NAME>  -c ApplicationDbContext  -o ./Data/Migrations/UserDB
dotnet ef migrations add <NAME>  -c ApplicationDbContext  -o ./Data/Migrations/UserDB
```
migrations 폴더에 추가되고 snapshot이 변경된다. 현재 디비에는 적용이 안되는 상태이다.

## update database - add new migration
```
update-database  -c ApplicationDbContext
dotnet ef database update -c ApplicationDbContext
```
디비에는 적용

## Empty migrations
```
add-migration <NAME>  -c ApplicationDbContext  -o ./Data/Migrations/UserDB
dotnet ef migrations add <NAME> -c ApplicationDbContext  -o ./Data/Migrations/UserDB
```
빈 마이그레이션을 생성한후 특별히 작업해야할 내용을 만들어준다.

## remove migration (Migration file 삭제)
```
remove-migration  -c ApplicationDbContext
dotnet ef migrations remove  -c ApplicationDbContext
```
* snapshot을 기존 버전으로 바꾸고 기존 마이그레이션 클래스도 지운다. 
* 최신 1개만 지운다.
* updatedatabase를 하면 디비에 적용될듯

## revert migration (마이그레이션 삭제와 동시에 DB 변경)
```
remove-migration -Force
dotnet ef database remove-migration  -Force
```

## sql 생성 - 디버깅, 확인
```
script-migration
dotnet ef migrations script
```

## seed data
DbContext에서 OnModelCreating 에서 hasdata를 사용하여 추가한다.

```cs
builder.Entity<Tenant>().HasData(
  new Tenant
  {
    Id = new Guid("a1fbde58-d642-4da9-889a-219343330f86"),
    Default = true,
    IsExternalLoginAvaliable = false,
    RootDomainId = new Guid("11fbde58-d642-4da9-889a-219343330f86")
  },
);
```

## Apply migrations at runtime
program.cs에서 host.Run() 앞에 다음처럼 넣으면된다.

```cs
using (var scope = host.Services.CreateScope())
{
  scope.ServiceProvider.GetService<ApplicationDbContext>().Database.Migrate();
}
...
host.Run();
```

## 아무런 변경없이 migration을 하면 seed data가 들어간다 왜?

일단 DateTime.now를 사용해서 매번 데이터가 바뀐다. 그래서 매번 바뀐 데이터가 생성  다 고정값을 줘도 계속 뭔가를 만든다.

확인해보니 AspNetUsers,AspNetRoles 값에 ConcurrencyStamp 를 항상 만들면서 넣어준다. 고정값이면 좋을거같은데. 이렇게 사용해도 별 문제는 없어보인다. 그대로 사용하자.


