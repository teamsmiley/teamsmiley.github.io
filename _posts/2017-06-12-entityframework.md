--- 
layout: post 
title: "entity framework cheet sheet" 
date: 2017-06-12 00:00  
author: teamsmiley 
tags: [entity]
image: /files/covers/blog.jpg
category: {entity}
---

# Entity Framework Cheet sheet 

* Guid를 id값으로 사용하려면 모델에서 다음처럼 생성 
```cs
...
Id = c.Guid(nullable: false, identity: true, defaultValueSql: "newsequentialid()"),
...
```

* enum을 사용해서 모델링을 해도 된다.

* 기존데이터가 잇는데 컬럼을 추가시 널이 안되게 해야한다.
이런경우는 false를 하면 에러가 난다..추가하는 컬럼이 null이 되기 때문이다.
다음처럼 null을 허용한후 데이터를 넣고 난후 null을 허용하지 않으면된다.
```cs
public override void Up()
{
    AddColumn("dbo.Products", "Location", c => c.String(nullable: true, maxLength: 50));
    Sql("UPDATE Products SET Location = 'core' WHERE(Location IS NULL)");
    AlterColumn("dbo.Products", "Location", c => c.String(nullable: false));
}
```
* colulm name을 변경하면 
FileName 을 Path로 바꾸고 마이그레이션을 하면 컬럼을 추가하고 sql을 이용해서 기존값을 새컬럼에 넣고 기존 칼럼을 drop하면된다.
근데 ef core로 오면서 다음처럼 자동으로 된다.
```cs
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.RenameColumn(
        name: "fileName",
        table: "Clips",
        newName: "Path");
}
```






