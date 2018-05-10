---
layout: post
title: 'Code Review - 1 ' 
author: teamsmiley 
date: 2018-05-10
tags: [code review]
image: /files/covers/blog.jpg
category: {program}
---

# code review - 초보자용

시간 날때마다 코드 리뷰를 해보고 결과를 적어둬야겟음. 

앵귤러 6로 작업중인 코드를 봤다. 

![]({{ site.baseurl }}/assets/code-review-1-before-1.PNG)

![]({{ site.baseurl }}/assets/code-review-1-before-2.PNG)

코드를 보는순간 뭐가 이상하다고 생각됬다. 왜이렇게 어렵지? 라는 생각이 제일 먼저 났다. 

다음 처럼 고쳐봤다. 

![]({{ site.baseurl }}/assets/code-review-1-after-1.PNG)

![]({{ site.baseurl }}/assets/code-review-1-after-2.PNG)

간단해졌는데. 이런걸 신입 개발자에게 어떻게 설명을 해야 할지는 잘 모르겟다.

```ts
update (bool:any,post:any)
```

일단 bool이 변수명이고 타입이 any다 잘못됬다. 
```ts
isPublish: boolean, post:Post 
```

이렇게 타입을 정해줘야한다. 

그리고 코드를 보니 단지 토글을 해주는것같다. 그럼 isPublish가 구지 들어갈 필요가없지 안나?

post.isPublish있기때문에 

```ts
  post.isPublish = !post.isPublish;
``` 
