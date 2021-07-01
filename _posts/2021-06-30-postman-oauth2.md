---
layout: post
title: 'Postman OAuth2 login'
author: teamsmiley
date: 2021-06-30
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# Postman OAuth2 Login

포스트맨으로 OAuth2 로그인을 해보자.

원하는 폴더에서 edit를 누르자.

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-16-57-57.png)

type : OAuth2를 선택
![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-16-58-53.png)

edit token configuration

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-17-00-39.png)

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-17-04-06.png)

정보를 잘 넣어주며 된다. identity server는 다음처럼 넣어주면 된다. 본인의 설정에 맞게 넣어주자.

- Callback URL : https://staging.xxxx.com/signin-callback

- Auth URL : https://auth.staging.xxxx.net/connect/authorize

- Access Token URL : https://auth.staging.xxxx.net/connect/token

이제 `Get New Access token` 버튼을 눌러보자.

로그인 창이 나오면 로그인 해본다.

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-17-11-50.png)

토큰을 받아오면

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-17-13-08.png)

![]({{ site_baseurl }}/assets/2021-06-30-postman-oauth2/2021-06-30-17-13-40.png)

use token을 누르면 저장된다.

## 브라우져를 이용하여 로그인하기

구글 로그인등에서 포스트맨 브라우저를 오래된 브라우저로 인식해 로그인이 안되는 경우가 있다. 그래서 웹브라우저를 통해서 로그인하는 방법을 찾아보았다.

일단 idp에서 다음 url을 redirect url에 추가해두어야한다.

- <https://oauth.pstmn.io/v1/callback>

이제 포스트맨에서 `Authorize using browser` 체크하면 브라우저가 실행된다. 로그인을 하면 포스트맨을 실행할거냐고 묻는다.

실행하면 토큰이 포스트맨으로 옮겨져 있다.
