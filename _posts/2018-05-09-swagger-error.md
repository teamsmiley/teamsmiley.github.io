---
layout: post
title: 'Swagger' 
author: teamsmiley 
date: 2018-05-06
tags: [Swagger,ConflictingActionsResolver]
image: /files/covers/blog.jpg
category: {swagger}
---

# Swagger NotSupportedException

## api에 Get함수가 두개가 있거나 하면 에러가 난다. 

http://localhost:62000/swagger

/swagger/v1/swagger.json 이 없다고 에럭 난다. 

http://localhost:62000/swagger/v1/swagger.json

을 해보면 다음처럼 에러가 난다. 

An unhandled exception occurred while processing the request.
NotSupportedException: HTTP method "GET" & path "boards/{boardname}/documents/{id}" overloaded by actions - API.Controllers.BoardDocumentsController.GetBoardDocument (rc-API),API.Controllers.BoardDocumentsController.GetBoardDocumentWithPublish (rc-API). Actions require unique method/path combination for Swagger 2.0. Use ConflictingActionsResolver as a workaround

해결 방법은 아래와 같다.

```cs
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new Info { Title = "Rendercore API", Version = "v1" });
    c.ResolveConflictingActions(apiDescriptions => apiDescriptions.First()); //추가 
});
```