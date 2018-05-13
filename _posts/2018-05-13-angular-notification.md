---
layout: post
title: 'Angular toastr notification' 
author: teamsmiley 
date: 2018-05-12
tags: [github]
image: /files/covers/blog.jpg
category: {program}
---

# ngx-toastr

요즘 디비 업데이트를 하거나 할때 화면에 알림 표시를 해주는 것을 본적이 있다. 
그래서 적용해 봤다. 

## install 

```bash
npm install ngx-toastr --save
```

## 설정 
angular-cli 프로젝트를 사용하므로 angular.json파일을 수정한다. 

```json
"styles": [
  "src/styles.css",
  "./node_modules/ngx-toastr/toastr.css"//추가 
],
```

app.module.ts에 다음 추가 한다. animation이 없는것과 있는것으로 구성할수 있는데 없는것을 사용하겠다. 

```ts
@NgModule({
  imports: [
    ToastNoAnimationModule,
        ToastrModule.forRoot({
          toastComponent: ToastNoAnimation,
          timeOut: 10000,
          positionClass: 'toast-top-right',
          preventDuplicates: true,
        }), // ToastrModule added
```

## 사용

```ts
import { ToastrService } from 'ngx-toastr';
@Component({
  ...
})
export class YourComponent {
  constructor(private toastr: ToastrService) {}
 
  showSuccess() {
    this.toastr.success('Hello world!', 'Toastr fun!');
  }
}
```
컴포넌트에 이렇게 추가하면된다. 

```ts
this.toastr.success('Hello world!', 'Toastr fun!');
this.toastr.error('Hello world!', 'Toastr fun!');
```

## Global Error Handler에 추가 

그냥 추가하면 글로벌 에러 핸들러에서는 동작하지 않는다. 

inject과 zone을 사용해야한다. 

```ts
import { ErrorHandler, Inject, Injector, isDevMode, NgZone } from '@angular/core';
import { ToastrService } from 'ngx-toastr';


export class AppErrorHandler implements ErrorHandler {

  constructor(
    @Inject(NgZone) private ngZone: NgZone,
    @Inject(Injector) private injector: Injector
  ) { }

  private get toastr(): ToastrService {
    return this.injector.get(ToastrService);
  }

  handleError(error: any): void {
    if (isDevMode()) {
      console.error(error.originalError || error);
    } 

    this.ngZone.run(() => {
      let errorTitle = 'Error';
      let errMsg = 'An unexpected error ocurred';
      this.toastr.error(errorTitle, errMsg);
      }
    })
  }
}
```

이렇게 injector를 인젝트한후 Injector.get으로 인젝트를 한후 ngZone.run에 추가하면된다. 

끝.




