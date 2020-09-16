---
layout: post
title: "Angular Data Bind (value)"
author: teamsmiley
date: 2020-09-16
tags: [angular, ionic]
image: /files/covers/blog.jpg
category: { programing }
---

# angular data bind에 대해서

어제 직원분이 뭐가 잘 안된다고해서 이어 받아서 처리한 내용을 정리해볼가 한다.

## 문제 발생

PaymentTypeNameComponent 라는 컴포넌트를 만드는데

enum은 다음처럼 있고

```ts
export enum EnumPaymentType {
  Cash,
  Card,
  Venmo,
}
```

ts는 다음처럼 생겼다.

```ts
import { Component, OnInit, Input } from '@angular/core';
import { FormArray, FormControl, FormGroup } from '@angular/forms';
import { EnumPaymentType } from '../../enums/enum-payment-type';

@Component({
@ -6,10 +7,24 @@ import { EnumPaymentType } from '../../enums/enum-payment-type';
  templateUrl: './payment-type-name.component.html',
})
export class PaymentTypeNameComponent implements OnInit {

  @Input('paymentTypeValue')
  set paymentTypeValue(value: number) {
    if (value !== null) {
      this.paymentType.setValue(value.toString()); //string으로 바구어 넣는다.
    }
  }

  paymentTypes = EnumPaymentType;

  paymentFormGroup: FormGroup = new FormGroup({
    paymentType: new FormControl(),
  });

  constructor() {}

  ngOnInit() {}

  get paymentType() {
    return this.paymentFormGroup.get('paymentType');
  }
}
```

html은 다음과 같다.

```html
<form [formGroup]="paymentFormGroup">
  <ion-radio-group formControlName="paymentType">
    <ion-item>
      <ion-label>Cash</ion-label>
      <ion-radio slot="start" value="0"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Card</ion-label>
      <ion-radio slot="start" value="1"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Venmo</ion-label>
      <ion-radio slot="start" value="2"></ion-radio>
    </ion-item>
  </ion-radio-group>
</form>
```

value부분을 잘 보기 바란다.

이렇게 되있으며 부모 컴포넌트에서 input으로 number type (0,1,2) 중에 하나를 던진다.

ts에서 숫자를 받아서 tostring()으로 바꿔서 formControlName="paymentType" 에 넣어서 화면에서 선택된 내용을 보여주는 것이다. 그런데 코드가 마음에 안들었다 enum이 있는데 안쓰고 "0", "1", "2" 라고 쓴다.

일단 tostring()하는 부분이 마음에 안들었다. 디비에 숫자가 잇어서 숫자가 그냥 넘어오는데 그냥 쓰면 되는데 싶었다.

ts에서 tostring을 제거하자.

```ts
@Input('paymentTypeValue')
  set paymentTypeValue(value: number) {
    if (value !== null) {
      this.paymentType.setValue(value); //string으로 바구어 넣는다.
    }
  }
```

이제 화면에서 확인해보는데 선택이 안된다.

확인해보니 value를 [value]로 사용하지 않아서 문제가 발생했다.

![]({{ site_baseurl }}/assets/2020-09-16-07-26-12.png)

```
value : html
[value] : angular bind
```

이렇게 보면 된다.

코드를 수정해보자.

```html
<form [formGroup]="paymentFormGroup">
  <ion-radio-group formControlName="paymentType">
    <ion-item>
      <ion-label>Cash</ion-label>
      <ion-radio slot="start" [value]="0"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Card</ion-label>
      <ion-radio slot="start" [value]="1"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Venmo</ion-label>
      <ion-radio slot="start" [value]="2"></ion-radio>
    </ion-item>
  </ion-radio-group>
</form>
```

![]({{ site_baseurl }}/assets/2020-09-16-07-26-36.png)

잘 선택된다.

## enum

이제 enum을 써보자.

```html
<form [formGroup]="paymentFormGroup">
  <ion-radio-group formControlName="paymentType">
    <ion-item>
      <ion-label>Cash</ion-label>
      <ion-radio slot="start" [value]="enumPaymentTypes.Cash"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Card</ion-label>
      <ion-radio slot="start" [value]="enumPaymentTypes.Card"></ion-radio>
    </ion-item>
    <ion-item>
      <ion-label>Venmo</ion-label>
      <ion-radio slot="start" [value]="enumPaymentTypes.Venmo"></ion-radio>
    </ion-item>
  </ion-radio-group>
</form>
```

화면도 잘 나오고 code도 깔끔해보인다.

이만 완료
