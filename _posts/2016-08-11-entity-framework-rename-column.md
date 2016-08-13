---
layout: post
title: 'Entity Framework Rename Column' 
author: teamsmiley 
date: 2016-08-11
tags: [Entity Framework]
image: /files/covers/blog.jpg
category: {entity framework}
---


Entity Framework Rename column

Renderer 를 RendererName 으로 변경해보자. 


```
public string Renderer { get; set; }
==>
public string RendererName { get; set; }

```


```
Add-Migration RendererRename
```
다음 코드가 생성이 된다.

``` 
public override void Up()
{
    AddColumn("dbo.Jobs", "RendererName", c => c.String());
    DropColumn("dbo.Jobs", "Renderer");
}
        
public override void Down()
{
    AddColumn("dbo.Jobs", "Renderer", c => c.String());
    DropColumn("dbo.Jobs", "RendererName");
}

```

update-database 해버리면 기존 데이터가 다 날라가 버린다.

그래서 중간에 한 줄을 추가한다.

``` 
public override void Up()
{
    AddColumn("dbo.Jobs", "RendererName", c => c.String());
    Sql("Update Jobs Set RendererName = Renderer");
    DropColumn("dbo.Jobs", "Renderer");
}
```

이제 기존데이터가 새 컬럼으로 저장되고 기존 컬럼은 지워진다.
