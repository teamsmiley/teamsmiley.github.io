---
layout: post
title: 'Angular gtag' 
author: teamsmiley
date: 2018-07-30
tags: [angular,gtag]
image: /files/covers/blog.jpg
category: {seo}
---

# Angular 6 에서 구글 애널리틱스 연결하기 

<https://github.com/codediodeio/angular-gtag>를 사용하자. 

## angular-gtag를 설치하자. 

npm install angular-gtag --save

## index.html수정
send_page_view를 false로 세팅하자.

```html
<head>
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-YOUR_TRACKING_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'UA-YOUR_TRACKING_ID', { 'send_page_view': false });
</script>
</head>
```

## app.module.ts 수정

```ts
import { GtagModule } from 'angular-gtag';

@NgModule({
  imports: [
    GtagModule.forRoot({ trackingId: 'UA-YOUR_TRACKING_ID', trackPageviews: true, debug: true })
  ]
})
```

## pageviews 보기 
패키지가 라우트 변경을 자동으로 인식한다. 그러므로 다음처럼 하면된다. AppComponent.ts 를 수정한다. 
```ts
export class AppComponent {
  constructor(gtag: Gtag) {}
}
```

## 이벤트 보내기
```ts
gtag.event('view_promotion');
```

이렇게 하면 이벤트가 동작한다. 

추가 데이터는 다음과 같다.

```ts
gtag.event('login', {
  method: 'Instagram',
  event_category: 'engagemnt',
  event_label: 'New user logged in via OAuth'
});
```

## Event Directive  디렉티브로 이벤트를 사용할수 있다 (버튼 클릭같은곳에 붙일수 있다는 이야기)
```html
<button gtagEvent trackOn="click">Track Me</button>
```

이렇게 하고 애널리틱스에서 리포트를 보면 데이터를 볼수 있다. 

## 추가 할 내용.
* 페이지는 이제 나온다 그런데 이 후로 뭐를 해야할지 모르겠음




