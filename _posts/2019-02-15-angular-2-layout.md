---
layout: post
title: 'angular 7 routing과 2개의 레이아웃 만들기' 
author: teamsmiley
date: 2019-02-14
tags: [devops]
image: /files/covers/blog.jpg
category: {angular}
---

# angular 7 routing and 2개의 레이아웃 만들기

현재 만들고 잇는 프로젝트가 로그인 전과 로그인 후가 레이아웃이 많이 다른다. 그래서 한번 만들어보았다. 


![]({{site_baseurl}}/assets/2019-02-15-16-51-35.png)

화면이 두개가 다른경우가 있다. 일단 로그인 전 화면과 로그인 후 화면은 화면구성이 아주 다르다.

그러므로 first 라는 화면과 second라는 화면이 라우팅에 따라 바뀌어야한다. 

```bash
# layout Module을 만들자. 
MODULE_NAME=layout-sample # 꼭 lowercase 단어는 - 로 구분
ng generate module ${MODULE_NAME} --routing=true

mkdir src/app/${MODULE_NAME}/components/
touch src/app/${MODULE_NAME}/components/.gitkeep
mkdir -p src/app/${MODULE_NAME}/services
touch src/app/${MODULE_NAME}/services/.gitkeep
mkdir -p src/app/${MODULE_NAME}/models
touch src/app/${MODULE_NAME}/models/.gitkeep


ng generate component /${MODULE_NAME}/components/first-layout --module ${MODULE_NAME} 
ng generate component /${MODULE_NAME}/components/second-layout --module ${MODULE_NAME} 

```


LayoutSampleRoutingModule에 다음 추가 
```ts
// const routes: Routes = []; 
const routes: Routes = [
  {
    path: 'first',
    component: FirstLayoutComponent,
  },
  {
    path: 'second',
    component: SecondLayoutComponent,
  },
];
```

이제 app.component에서 임포트해서 사용한다.

```ts
//...
  imports: [
    LayoutSampleModule,
//...    
export class AppModule { }
```

ui에서 확인해보자. 잘된다. 

### 추가 문제 

이제 레이아웃에 따라 화면을 바꿔서 보여줘야한다. 

first layout으로 테스트해본다.

Hello 컴포넌트와 HI 컴포넌트를 만들어서 first layout에서 /first/hello / first/hi에 따라서 컴포넌트를 바꿔보자.

```bash
ng generate component /${MODULE_NAME}/components/first-layout/hello --module ${MODULE_NAME} 
ng generate component /${MODULE_NAME}/components/first-layout/hi --module ${MODULE_NAME}
```

first-layout.html을 수정하자.
```html
<router-outlet></router-outlet>
```

라우팅을 세팅하자.

```ts
const routes: Routes = [
  {
    path: 'first',
    component: FirstLayoutComponent,
    children: [
      {
        path: 'hello',
        component: HelloComponent,
      },
      {
        path: 'hi',
        component: HiComponent,
      }
    ]
  }
];
```

결과

![]({{site_baseurl}}/assets/2019-02-15-16-49-51.png) 
![]({{site_baseurl}}/assets/2019-02-15-16-50-17.png)

잘된다.





