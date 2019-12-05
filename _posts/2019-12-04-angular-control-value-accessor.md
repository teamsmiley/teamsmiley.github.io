---
layout: post
title: 'Angular Control Value Accessor' 
author: teamsmiley
date: 2019-12-04
tags: [angular]
image: /files/covers/blog.jpg
category: {angular}
---

# Angular Control Value Accessor
커스텀 컴포넌트를 폼에서 사용하려면 여러가지 문제에 도달하게 된다. 컴포넌트가 기본 컴포넌트 input select 등과 다르게 움직이기때문이다. Control Access Value를 구현하면 커스텀 컴포넌트도 인풋을 기본 컴포넌트처럼 사용할수 있게 된다. (validation, disable, etc )

## Control Access Value를 구현

4가지 기본 함수를 구현해야한다.
```ts
public writeValue(obj: T) //=> 값을 html에 넣는다.

public registerOnChange(fn: any) //=> 컴포넌트 값이 바뀌면 실행

public registerOnTouched(fn: any) //=>컴포넌트가 터치가 되면

public setDisabledState?(isDisabled: boolean) //=> 컴포넌트가 disable이 될때
```

대부분 비슷하므로 base class 를 구현해두고 상속받아서 쓰자.

BaseControlValueAccessor.ts
```ts
import { ControlValueAccessor } from '@angular/forms';

export class BaseControlValueAccessor<T> implements ControlValueAccessor {

  // public onChange(newVal: T) {}
  // public onTouched(_?: any) {}

  value: T;
  onChange: () => void;
  onTouched: () => void;
  disabled: boolean;

  public writeValue(obj: T): void {
    console.log('writeValue:', typeof (obj), ":", obj)
    this.value = obj;
  }
  public registerOnChange(fn: any): void {
    console.log('registerOnChange')
    this.onChange = fn;
  }
  public registerOnTouched(fn: any): void {
    console.log('registerOnTouched')
    this.onTouched = fn;
  }

  public setDisabledState?(isDisabled: boolean): void {
    console.log('setDisabledState:', isDisabled)
    this.disabled = isDisabled;
  }
}
```
custom-select-form.component.ts
```ts
import { Component, Input, forwardRef } from '@angular/core';
import { NG_VALUE_ACCESSOR } from '@angular/forms';
import { BaseControlValueAccessor } from '@shared/common/BaseControlValueAccessor';

@Component({
  selector: 'custom-select-form',
  templateUrl: './custom-select-form.component.html',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      multi: true,
      useExisting: forwardRef(() => CustomSelectFormComponent)
    }
  ]
})
export class CustomSelectFormComponent extends BaseControlValueAccessor<string> {
  @Input('items') items: any;
  @Input('label') label: any;

  constructor() {
    super();
  }
  ngOnInit() { }
}
```
custom-select-form.component.html
```html
<mat-form-field appearance="standard">
  <mat-label *ngIf="label">{{label}}</mat-label>
  <mat-select [(value)]="value" (selectionChange)="onChange($event.value)" (blur)="onTouched()">
    <mat-option *ngFor="let item of items" [value]="item.value">
      {{ item.title }}
    </mat-option>
  </mat-select>
</mat-form-field>
```

## 컴포넌트를 사용

```html
<div fxFlex.gt-sm="100" fxFlex.gt-xs="100" fxFlex="100">
  <form [formGroup]="applicantStepFormGroup">
    <custom-select-form [items]="applicantStepStatus" [label]="'Status'" formControlName="status">
    </custom-select-form>
  </form>
</div>
```

```ts
export class AgreementComponent implements OnInit {

this.formGroup = new FormGroup({
  status: new FormControl({ value: null, disabled: true }, [Validators.required]) 
});

}
```

* html에 `[disabled]=disable` 로 작업시 추천방식이 아니라는 워닝. 추천코드를 보여줌 위 코드는 그걸 적용한것임.
* `{ value: null, disabled: true }` 이거를 `{ disabled: true }` 이걸로 하면 disable코드가 동작하지 않음.
* `{ value: null, disabled: true }` 이코드를 FormControl에 넣거나 ngOnInit에서 `this.applicantStepFormGroup.get('status').disabled();`를 호출해주면 되는듯.

## 이벤트 처리

기존과 다르게 event를 잡아오는 방식이 좀 다르다. `valueChanges` 를 사용한다.

* 부모 컴포넌트 ngOnInit 에서 다음처럼

```ts
// form 전체의 값변화를 받는다.
this.myForm.valueChanges.subscribe(val => {
...
});

// form control의 값 변화를 받는다.
this.myForm.get('name').valueChanges.subscribe(val => {
  console.log(val);
});
```

## 서버에서 값을 가져와서 폼에 set하기
```ts
this.formGroup.patchValue(product); // product 서버에서 가져온 값
```



