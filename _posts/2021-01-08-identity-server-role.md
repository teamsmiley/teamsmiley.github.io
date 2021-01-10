---
layout: post
title: "Identity Server4 에서 access token에 Role 포함하기"
author: teamsmiley
date: 2021-01-08
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# Identity Server4 에서 access token에 Role 포함하기

id server에서 dotnet membership을 쓰는데 token에 role을 포함해야하는 경우가 생겼다.

진행해보자.

## config.cs

```cs
public static IEnumerable<IdentityResource> GetIdentityResources()
{
  return new IdentityResource[]
  {
    ...
    new IdentityResource // 추가
    {
        Name = "roles",
        DisplayName = "Roles",
        UserClaims = { JwtClaimTypes.Role }
    }
  };
}

public static IEnumerable<ApiResource> GetApis()
{
  return new ApiResource[]
  {
    ...
    new ApiResource("api", "API", new List<string>() { JwtClaimTypes.Role }){ // 추가
      Scopes = new []{
        "api",
      }
    },
  };
}

 public static IEnumerable<Client> GetClients()
    {
      return new[]
      {
        new Client
        {
          ...
          AllowedScopes = {
            ...
            "roles" // 추가
          },

        },
```

기본작업은 끝낫다. 이제 dotnet membership에서 롤을 읽어와서 토큰에 넣어주면된다.

두가지 방식이 있다.

- profile service를 만들어서 처리
- UserClaimsPrincipalFactory를 구현한 클래스를 만들어서 di로 처리

UserClaimsPrincipalFactory를 구현하면

```cs
namespace App.Services
{
    public class ClaimsFactory : UserClaimsPrincipalFactory<AppUser>
    {
        private readonly MyDbContext _context;
        private readonly UserManager<AppUser> _userManager;

        public ClaimsFactory(
            UserManager<AppUser> userManager,
            IOptions<IdentityOptions> optionsAccessor,
            MyDbContext context) : base(userManager, optionsAccessor)
        {
            _context = context;
            _userManager = userManager;
        }

        protected override async Task<ClaimsIdentity> GenerateClaimsAsync(AppUser user)
        {
            var identity = await base.GenerateClaimsAsync(user);
            var roles = await _userManager.GetRolesAsync(user);

            identity.AddClaims(roles.Select(role => new Claim(JwtClaimTypes.Role, role)));

            return identity;
        }
    }
}
```

startup.cs에서

```cs
public void ConfigureServices(IServiceCollection services)
{
  services.AddClaimsPrincipalFactory<ClaimsFactory>();
}
```

참고 : <https://themisir.com/adding-role-claim-to-identity-server-4/>

## profile service를 사용하면

ProfileService.cs

```cs
public async Task GetProfileDataAsync(ProfileDataRequestContext context)
{
  var sub = context.Subject.GetSubjectId();

  var user = await _userManager.FindByIdAsync(sub);

  var roles = await _userManager.GetRolesAsync(user);

  context.IssuedClaims.AddRange(roles.Select(role => new Claim(JwtClaimTypes.Role, role)));
}
```

이런식으로 롤을 가져와서 토큰에 추가해주면 된다.

이제 token을 확인해보면 role이 포함된것을 알수 있다.

## api

이제 api에서 role 을 줘서 api를 보호해줘야한다.

```cs
[Authorize(Roles = "Admin")]
[Route("advertise-details")]
public class AdvertiseDetailsController : ApiController
...
```

완성
