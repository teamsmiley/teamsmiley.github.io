---
layout: post
title: 'Angular Form Array' 
author: teamsmiley
date: 2019-11-11
tags: [angular]
image: /files/covers/blog.jpg
category: {dev}
---

# Angular Form Array

폼에서 Array를 써서 동적으로 컴포넌트를 ADD/DELETE 해야 할 경우가 생겼다. 그런데 잘 안되서 정리 

Ts에서 다음처럼 코딩햇다. 

```ts
export class NestedFormArray {
  form = new FormGroup({
    cities: new FormArray([
      new FormControl('SF'),
      new FormControl('NY'),
    ]),
  });

  get cities() { 
    return this.form.get('cities') as FormArray; 
  }

  addCity() { this.cities.push(new FormControl()); }
```

Html에서는 다음처럼
```html
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <div formArrayName="cities">
    <div *ngFor="let city of cities.controls; index as i">
      <input formControlName="{{i}}" placeholder="City">
    </div>
  </div>
  <button>Submit</button>
</form>

<button (click)="addCity()">Add City</button>
```

페이지가 로딩될때 2개의 도시가 나오고 Add city버튼이 눌릴때마다 추가하게 된다. 

그런데 에러가 나기 시작햇다. 
```
Error: Cannot find control with path: 'times -> i'
```

![]({{site_baseurl}}/assets/angular-properties-bind.png)

원인은 내 코드였다.

```html
<time-select formControlName="i"></time-select>
```

위 샘플 코드와 조금 달랐다.  "{{i}}" 가 없었다. 하루를 낭비후 {{}} 넣고 처리해보니 에러가 안됨.

다른 샘플 코들에서는 {{}} 가 없이 사용되기도 하였다. 확인을 해보니 

```html
<time-select [formControlName]="i"></time-select>
```

이렇게 braket을 붙여주면 동작한다.

그래서 두개가 무슨 차이가 있는지 궁금했다.

<https://app.pluralsight.com/guides/property-binding-angular>

여기를 참고했다 중간에 보면 다음글이 보인다.

Don't forget to put brackets around the property during property binding. Brackets will tell Angular to evaluate the template expression. If you forget to use brackets, then Angular will treat the string as a constant and initialize the target property with that string. It will not evaluate the string.

다음부터는 formControlName등에는 꼭 []를 붙여서 쓰는 규칙을 만들어야겟다.









