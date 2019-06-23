---
layout: post
title: 'OAuth 2 Client Credential Flow' 
author: teamsmiley
date: 2019-06-28
tags: [devops]
image: /files/covers/blog.jpg
category: {oauth}
---
# OAuth 2 Client Credential Flow (Server to Server or Service Account)

Oauth를 공부한지 한 2년이 되가는데 제일 이해가 안되는 부분이 이부분이 였다.

보통 server to server에 사용된다고 나와있기는 했다. 

예를 들면 어떤 데몬을 만들어서 5분에 한번씩 api서버를 호출하는 console프로그램을 만든다고 해보자. 매뉴얼대로 해보면 인증은 되나 유저 정보가 없어서 클라이언트(프로그램)는 인증이 되나 내용을 가져올수가 없어서 이걸 왜 쓰는지도 모르고 넘어갔다. 인터넷에 몇번을 찾아봐도 ( 2년동안 1달 간격으로 이 이슈를 검색해봣던거 같다.) 내용 파악이 안됬고 책을 구매해서 보기도 햇는데 이해가 안됬다. 

그러다 이번에 정리가 좀 된것같아 적어보겠다. 이건 정말 개인적인 생각이므로 따라하실분은 다른곳도 많이 보고 난후 결정하기 바란다. 

일단 Client credential flow를 보자. 

<https://oauth.net/2/grant-types/client-credentials/>

<https://tools.ietf.org/html/rfc6749#section-4.4>

보통 client는 브라우저나 console application등 프로그램을 이야기하고  고객은 user라고 한다. 

다음을 보자.

```
The client can request an access token using only its client
   credentials (or other supported means of authentication) when the
   client is requesting access to the protected resources under its
   control, or those of another resource owner that have been previously
   arranged with the authorization server (the method of which is beyond
   the scope of this specification).

   The client credentials grant type MUST only be used by confidential
   clients.

     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+

                     Figure 6: Client Credentials Flow

   The flow illustrated in Figure 6 includes the following steps:

   (A)  The client authenticates with the authorization server and
        requests an access token from the token endpoint.

   (B)  The authorization server authenticates the client, and if valid,
        issues an access token.

4.4.1.  Authorization Request and Response

   Since the client authentication is used as the authorization grant,
   no additional authorization request is needed.

   For example, the client makes the following HTTP request using
   transport-layer security (with extra line breaks for display purposes
   only):

     POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=client_credentials

   The authorization server MUST authenticate the client.

4.4.3.  Access Token Response

   If the access token request is valid and authorized, the
   authorization server issues an access token as described in
   Section 5.1.  A refresh token SHOULD NOT be included.  If the request
   failed client authentication or is invalid, the authorization server
   returns an error response as described in Section 5.2.

   An example successful response:

     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "example_parameter":"example_value"
     }
```

정리하면 client id / client secret을 보내서 인증서버에서 인증해주는 방법이다. 

grant_type은 client_credentials을 보내준다. 

그러면 결과값으로 토큰이 넘어온다. 이 토큰은 clientid는 있으나 기타 정보가 하나도 없다 예를들면 userid나 role등이 없다. 

여기서 항상 막혓다. 

그러나 어제 갑자기 뭔가가 생각이 났다.

유저를 만들고 유저가 client id/secret을 만들면 결국 client가 user정보를 다 가지고 있는것과 같다.

결과적으로 client id/secret을 만들때 user id (sub)정보를 같이 넣어주고  클라이언트 인증시 토큰에 강제로 sub을 추가해주면 되지 않을가? 

identity server 4를 사용하는 나는 일단 두가지를 해야햇다 

client id/sercret를 만들때 user정보를 넣어주는것이다. 확인해보니 clientProperties라는 테이블이 있다. 

예를들면 다음과 같다.
```cs
new Client
{
    ClientId = "12341234123",
    ClientName = "Client Credentials Client",
    AllowedGrantTypes = GrantTypes.ClientCredentials,
    ClientSecrets = { new Secret("secret".Sha256()) },
    AllowedScopes = {
      IdentityServerConstants.StandardScopes.OpenId,
      IdentityServerConstants.StandardScopes.Profile,
      "roles"
    },
    Properties= new Dictionary<string, string>(){
      {"sub","c69e3d32-7d75-481e-b0a5-f1a057f50cd5"},
      {"role","Admin"}
    },
    AccessTokenLifetime = 3600*24*30 //토큰 유효시간을 길게 server to server 라서 
},
```

이렇게 프로퍼티에 넣어 주고 나면 이제 인증할때 token에 userid(sub)을 포함시켜줘야한다.

ICustomTokenRequestValidator 을 구현한 클래스를 하나 만들어 주자.

```cs
using System.Security.Claims;
using System.Threading.Tasks;
using IdentityServer4.Validation;

public class ClientCredentialRequestValidator : ICustomTokenRequestValidator
{
    public async Task ValidateAsync(CustomTokenRequestValidationContext context)
    {
        var client = context.Result.ValidatedRequest.Client;

        if (client.ClientId == "client")
        {
            context.Result.ValidatedRequest.ClientClaims.Add(new Claim("sub", client.Properties["sub"]));
            context.Result.ValidatedRequest.ClientClaims.Add(new Claim("role",client.Properties["role"]));

            // don't want it to be prefixed with "client_" ? we change it here (or from global settings)
            context.Result.ValidatedRequest.Client.ClientClaimsPrefix = "";
        }
    }
}
```

startup.cs에서 설정을 추가해준다.
```cs
var builder = services.AddIdentityServer()
      ...
      .AddAspNetIdentity<ApplicationUser>()
      .AddCustomTokenRequestValidator<ClientCredentialRequestValidator>()
      ;
```

이렇게 하면 벨리데이터에서 token 에 sub을 넣어서 리턴해준다.

완료.






