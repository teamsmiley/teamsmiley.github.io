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

시간 날 때마다 코드 리뷰를 해보고 문서를 작성하자. 

앵귤러 6.0 작업 중 

![]({{ site.baseurl }}/assets/code-review-1-before-1.PNG)

![]({{ site.baseurl }}/assets/code-review-1-before-2.PNG)

코드를 보는 순간 뭐가 이상하다고 생각됐다. 왜 이렇게 어렵지? 라는 생각이 제일 먼저 났다. 

다음처럼 고쳐봤다. 

![]({{ site.baseurl }}/assets/code-review-1-after-1.PNG)

![]({{ site.baseurl }}/assets/code-review-1-after-2.PNG)

간단해졌는데. 이런 걸 신입 개발자에게 어떻게 설명을 해야 할지는 잘 모르겠다.

```ts
update (bool:any,post:any)
```

일단 bool이 변수명이고 타입이 any다 

잘못도ㅒㅣㅓㅗㅑ다. 

```ts
isPublish: boolean, post:Post 
```

이렇게 타입을 정해줘야한다. 

그리고 코드를 보니 단지 토글을 해 준다. 그럼 isPublish가 굳이 들어갈 필요가 없는듯

post.isPublish 가 있기 때문에 

```ts
  post.isPublish = !post.isPublish;
```

이렇게 해주면 아규먼트로 넘기지 않아도 된다. 

긍정적인 코딩으로 진행하기 때문에 일단 화면을 업데이트하고 디비업데이트를 나중에 하는것으로 순서를 바꿔야한다. 

데이터베이스가 성공이 오면 화면을 바꾸게 코딩이 되어있다. 위로 올리자. 그러고 나니 array에서 인덱스를 찾아와서 뭔가를 하는데 이건 안해도 될 로직이다. 왜냐면 post가 벌써 모든 정보를 가지고 있기 때문에 posts를 가져와서 다시 데이터를 가져오지 않아도 된다.  

화면에서 data-toggle이 필요 없을 듯 해서 지우고 확인하니 잘 동작함.

모든게 중복이었다. 중복을 제거해야 하는데 이게 중복이라고 설명을 어떻게 해야 할지 고민해야 한다. 



