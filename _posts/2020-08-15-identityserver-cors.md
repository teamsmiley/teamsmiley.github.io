---
layout: post
title: 'identityserver4 cors setting' 
author: teamsmiley
date: 2020-08-15
tags: [identity server]
image: /files/covers/blog.jpg
category: {identity server}
---

# identity server - cors setting

id server에서 자꾸 에러가 나서 뭐가 안되나 싶어서 확인해보았다.

## 어떤 cors도 허용

startup에 코드를 추가해주면된다.
```cs
public void ConfigureServices(IServiceCollection services)
{
  services.AddCors(options =>
  {
    options.AddPolicy("AllowAll", builder =>
    {
      builder.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod()
              ;
    });
  });
...
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
  app.UseCors("AllowAll");
}
```

기본적으로 이렇게 해두면 전체 cors가 허용된다.

## 클라이언트마다 cors가 바뀌게 적용하려면?

config에 클라이언트마다 추가로 적용할수가 있게 되있다.
```cs
new Client
{
  ClientName = "x user app",
  ClientId = "x-user-app",
  RequireClientSecret = false,
  AllowedGrantTypes = GrantTypes.Code,
  RequirePkce = true,
  AllowAccessTokensViaBrowser = true,
  RequireConsent = false,
  AccessTokenLifetime = 36000,
  AllowOfflineAccess = true,

  RedirectUris =
  {
      "http://localhost:11000/signin-callback",
  },
  PostLogoutRedirectUris = new List<string>()
  {
      "http://localhost:11000/signout-callback",
  },
  AllowedCorsOrigins = {
      "http://localhost:11000",
  },
  AllowedScopes = {
      IdentityServerConstants.StandardScopes.OpenId,
      IdentityServerConstants.StandardScopes.Profile,
      IdentityServerConstants.StandardScopes.Email,
      "x-user-api"
  },
},
```

이렇게 AllowedCorsOrigins 를 적용해버린 클라이언트에는 startup에서 만든것은 무시되고 default가 적용이 된다. 그러므로 허용된 cors만 허가되게 된다.

주석처리해버리면(설정을 안하면) 모든 cors를 허용한다.






