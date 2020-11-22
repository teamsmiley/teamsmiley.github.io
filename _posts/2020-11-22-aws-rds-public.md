---
layout: post
title: "aws rds public access"
author: teamsmiley
date: 2020-11-22
tags: [devops]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# aws rds public access

aws rds 서비스를 외부에서 엑세스가 가능하게 만들기

## db instance가 외부 접속 가능하게 설정

1. aws console > rds > db instance > 원하는 db instance 선택후 modify
1. connectivity > Additional connectivity configuration
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-37-08.png)
1. save

## vpc에서 라우팅을 internet gateway로 설정

1. aws console > rds > db instance > 원하는 db instance 선택후 > vpc 선택
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-40-03.png)
1. click routing table
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-41-56.png)
1. edit routes
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-43-35.png)
1. 0.0.0.0/0 internet gateway 추가
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-44-59.png)

## security group 수정

1. aws console > rds > db instance > 원하는 db instance 선택후 > security group 선택
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-40-03.png)
1. inbound > edit >> add myip
   ![]({{ site_baseurl }}/assets/rds/2020-11-22-06-48-10.png)
1. save

## 확인

이제 end point 주소로 확인을 해보면 된다.

```
telnet xxxxxx 3306
```

동작하면 외부에 오픈 성공
