---
layout: post
title: 'Angular 4 i18n with ngx-translate' 
author: teamsmiley 
date: 2018-04-22
tags: [angular]
image: /files/covers/blog.jpg
category: {program}
---

# angular i18n 지원 

angular 6와 ngx를 사용하지않고 기본 패키지를 사용해서 i18n을 구현은 다음 링크를 보세요

<https://teamsmiley.github.io/2018/07/29/angular6-i18n-ssr-aot/>

## i18n 지원 

angular 4.2를 기준으로 사용하는 패키지가 다른다. 4.3부터는 최신버전 4.2까지는 하위버전 ..

### 4.2 이하 
* 패키지 추가 

cmd (administrator)

```
npm install @ngx-translate/core@^7.2.2 -D 
npm install @ngx-translate/http-loader@0.1.0 -D
npm install @types/node -D
npm install @ngx-cache/core@4.0.1 -D
npm install @ngx-universal/translate-loader@4.0.1 -D
```

webpack.vendor.js 설정 추가 
```ts
const treeShakableModules = [
    '@angular/animations',
    '@angular/common',
    '@angular/compiler',
    '@angular/core',
    '@angular/forms',
    '@angular/http',
    '@angular/platform-browser',
    '@angular/platform-browser-dynamic',
    '@angular/router',
    '@ngx-translate/core',
    '@ngx-translate/http-loader',
    'zone.js',
];
```

### 클라이언트 랜더링

app.shared.module 에 다음 추가

```ts
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { BrowserModule } from "@angular/platform-browser";

//AoT requires an exported function for factories
export function createTranslateLoader(http: Http,) {
    return new TranslateHttpLoader(http);
}

 imports: [
        BrowserModule,
        CommonModule,
        HttpModule,
        FormsModule,
        TranslateModule.forRoot({
            loader: {
                provide: TranslateLoader,
                // useFactory: HttpLoaderFactory,
                useFactory: (createTranslateLoader), //AoT
                deps: [Http]
            }
        }),
        ...
exports: [
        TranslateModule
    ]      
```

### 서버 사이드 랜더링 

app.server.module 에 다음 추가 

```ts
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { UniversalTranslateLoader } from '@ngx-universal/translate-loader';
import { PLATFORM_ID } from '@angular/core';

export function createTranslateLoader(platformId: any, http: Http): TranslateLoader {
    const browserLoader = new TranslateHttpLoader(http);
    return new UniversalTranslateLoader(platformId, browserLoader, './wwwroot/assets/i18n');
}

 imports: [
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: (createTranslateLoader),
        deps: [PLATFORM_ID, Http]
      }
    }),

```

컴포넌트 추가

navmenu.html에 다음 추가

```html
                <li [routerLinkActive]="['link-active']">
                    <a [routerLink]="['/home']">
                        <span class='glyphicon glyphicon-home'></span> {{ 'NAV.HOME' | translate }}
                    </a>
                </li>
                <li [routerLinkActive]="['link-active']">
                    <a [routerLink]="['/blog/posts']">
                        <span class='glyphicon glyphicon-education'></span> {{ 'NAV.BLOG' | translate }}
                    </a>
                </li>

            </ul>
        </div>

        <select #langSelect (change)="translate.use(langSelect.value)">
            <option *ngFor="let lang of translate.getLangs()" [value]="lang" [selected]="lang === translate.currentLang">{{ lang }}</option>
        </select>
    </div>
</div>

```
navmenu.ts에 다음 추가 
```ts
import { TranslateService } from '@ngx-translate/core';

export class AAA {

    param = {value: 'world'};

    constructor(public translate: TranslateService) {
        translate.addLangs(["en", "ko"]);
        // this language will be used as a fallback when a translation isn't found in the current language
        translate.setDefaultLang('en');

        // the lang to use, if the lang isn't available, it will use the current loader to get them
        translate.use('en');
        
        //let browserLang = translate.getBrowserLang();
        //console.log("browser lang : " + browserLang); 
       //translate.use(browserLang.match(/en|ko/) ? browserLang : 'en');
    }
}
```

wwwroot하위에 다음을 만들어 준다. 

assets/i18n/ko.json
```json
{
    "HOME": {
        "HELLO": "안녕 {{value}}"
    }
}
```
assets/i18n/en.json
```json
{
    "HOME": {
        "HELLO": "hello {{value}}"
    }
}
```

### 4.3 이상 
* 패키지 추가 

```
npm install @ngx-translate/core --dev
npm install @ngx-translate/http-loader --dev
```

그리고  아래처럼 
```ts
export function createTranslateLoader(http: Http,) {
```
Http를 쓰지말고 HttpClient를 사용하면된다.

이러면된다..

### 잘 안되는 문제 


* translate.use(browserLang.match(/en|ko/) ? browserLang : 'en'); 이거 에러..왜? //todo
이건 서버사이드랜더링에서 브라우저가 없기때문에 에러가 난다..일단 주석처리 
서버 사이드 랜더링을 끄고 나면 별 문제없이 잘 동작한다..
//todo 그럼 서버사이드 랜더링에서는 기본값으로 하고 ...로컬에 가져온후에 브라우저 언어로 바꿔주는걸 해야할듯...

* 참고 웹사이트 
<https://medium.com/letsboot/translate-angular-4-apps-with-ngx-translate-83302fb6c10d>

## ngx-translate 가 서버사이드 랜더링에서 url을 상대경로로 처리

* url을 넣어주면 동작하게까지는 했음.

* 찾아보니 universal translate라는게 잇음  

4.0.1버전 내용을 확인해야함..버전이 다른면 다른 페이지를 봐야함.

<https://github.com/fulls1z3/ngx-translate/tree/4.x.x/packages/%40ngx-universal/translate-loader>

위에서 수정된 코드로 진행



