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

정보를 잘 넣어주며 된다. identity server는 다음처럼 넣어주면 된다.

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
