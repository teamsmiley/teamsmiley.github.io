---
layout: post
title: 'angular - data를 두개이상의 컴포넌트에서 공유하기' 
author: teamsmiley
date: 2020-06-23
tags: [angular]
image: /files/covers/blog.jpg
category: {angular}
---

# angular에서 두개 이상의 컴포넌트에서 데이터 공유하기

## Service

서비스를 하나 만들고 private으로 subject를 만든다. (rxjs) 

subject를 asObservable로 하는 변수를 하나 만든다. 

변수에 값을 세팅하는 함수도 하나 만든다. next를 하면 subscribe하고 있는 모든 컴포넌트에 전달이 된다. 

```ts
export class CartService {

  private _cartCountSource = new Subject<number>();
  cartCountSource$ = this._cartCountSource.asObservable();

  cartCount(count: number){
    this._cartCountSource.next(count);
  }
}
```

## 데이터를 보낼 컴포넌트 
서비스를 injection하고  함수를 호출한다. 

```ts
constructor(
    private service: CartService,
)

this.service.cartCount(this.cartCount);

```

## 데이터를 받을 컴포넌트
ngOnInit에서 subscribe를 해두면 된다.

그러면 service에서 next로 값을 넘겨주니 받아서 사용만 하면된다. 

```ts
ngOnInit() {
  this.cartService.cartCountSource$.subscribe(count => {
    this.cartCount = count;
  });
}
```


## 결론

redux를 통해서 하려고 햇던 기능을 이걸로 대체했다. redux가 좀 과한듯한 느낌이있엇다. 

언제나 redux를 쓸만한 프로젝트를 해보나..

## 참고

* <https://www.youtube.com/watch?v=oj6Tae2oSo0&list=PLC3y8-rFHvwgKhaLU8GTyF-5Bb8qT-wzV&index=14>
* <https://www.youtube.com/watch?v=eqz35AQoVcs&list=PLC3y8-rFHvwgKhaLU8GTyF-5Bb8qT-wzV>