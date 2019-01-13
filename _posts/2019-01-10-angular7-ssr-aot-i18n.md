---
layout: post
title: 'Angular 7 i18n SSR ' 
author: teamsmiley 
date: 2018-07-29
tags: [angular,ssr,i18n]
image: /files/covers/blog.jpg
category: {program}
---

# Angular 6 i18n with SSR 

## ng cli를 설치한다. 
```
npm install -g @angular/cli
```

## new project 생성 

```bash
ng new i18n-sample --routing
cd i18n-sample
npm i 
ng serve -o  //화면 잘 나오는지 확인 이건 ssr적용이 안된것이다. 
```
기본 프로젝트가 실행됬다. 깃으로 파일 변경여부를 잘 확인 해가면서 보자.

## 새 컴포넌트 추가

```bash
ng g c users
# ng g c users --module app
```

vi app-routing.module.ts
```ts
import { UsersComponent } from './users/users.component';
const routes: Routes = [
...
{ path: "users", component: UsersComponent },
...
```

vi app.component.html 에 다음 추가 
```html
<a [routerLink]="['/users']">users</a>
```

## SSR 을 설정하자

https://github.com/maciejtreder/ng-toolkit

```
ng add @ng-toolkit/universal
npm run build:prod
npm run server
```

설명을 조금 하면 ng add @ng-toolkit/universal 를 하면 기존에 하나씩 파일을 추가해주던 것을 자동으로 해준다. 여러 매뉴얼을 보면 이 자동 작업을 수동으로 하는 매뉴얼들이 있다 필요하면 참고하기 바란다. 

관련 파일들이 설치되면 npm run build:prod 를 실행하는데 이것은 package.json에서 확인가능하다 코드는 다음과 같다.
```json
"build:server:prod": "ng run renderfarm-app:server && webpack --config webpack.server.config.js --progress --colors",
"build:browser:prod": "ng build --prod",
"build:prod": "npm run build:server:prod && npm run build:browser:prod",
```

build:prod 를 실행하면 npm run build:server:prod (서버)이것이 실행되고 그다음에 npm run build:browser:prod (로컬)이것이 실행된다. 빌드 된 파일을 dist에 browser 와 server라는 폴더를 생성해서 거기에 넣어둔다.

--prod는 기본으로 --aot를 가지고 있다. angular.json에서 확인 가능

http://localhost:8080/에 접속해서 소스보기를 하면 ssr이 된것을 알수 있다.  컨텐츠 내용이 보이면 성공 

## 이제 실서버에 올려서 확인을 해보자. docker and nginx 

vi dockerignore
```
node_modules
dist
e2e
doc
README.md
.git
```

vi Dockerfile

```ini
FROM node:10 as node
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build:prod
EXPOSE 80
CMD [ "node", "local.js" ]
```

도커 빌드후 실행 

처음 요청시 소스코드를 보면 전체 내용이 있는것을 알수 있다. 그러나 링크를 클릭하면 페이지가 다 로드되는것이 아니라 싱글페이지앱으로 동작한다. 

결론은 첫 요청은 server side rendering으로 돌고  그 후 요청은 전부 spa로 동작한다.  seo가 잘 될듯.




## i18n 

앵귤러 빌드를 언어별로 해서 각각의 dist폴더를 만들어서 넣어서 고객의 언어에 따라서 /ko 등으로 보내주려고 한다.

### 필요한 화면에 표시를 한다.

src/app/app.component.html에서 다국어를 원하는 곳에 i18n을 붙인다. 

```html
<h2 i18n >Here are some links to help you start: </h2>
```

### generate language file 

```
ng xi18n
mkdir src/locale
mv src/messages.xlf src/locale
cp src/locale/messages.xlf src/locale/messages.ko.xlf
```

기본 파일을 앵귤러가 자동으로 생성해준다.
일단 기본파일에 ko를 붙여서 한글 파일을 만들어보자.
이걸 열어보면 <source>XXX</source>라는 부분이 있다.이부분 아래에 <target>XXX</target> 을 추가하면 된다. 

messages.ko.xlf 
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
  <file source-language="en" datatype="plaintext" original="ng2.template">
    <body>
      <trans-unit id="1e9a15da9ecb3574be8b466c285ed4aca1d89e4b" datatype="html">
        <source>
    Welcome to <x id="INTERPOLATION" equiv-text="{{ title }}"/>!
  </source>
  <target>
    환영합니다 <x id="INTERPOLATION" equiv-text="{{ title }}"/>!
  </target>
        <context-group purpose="location">
          <context context-type="sourcefile">app/app.component.html</context>
          <context context-type="linenumber">3</context>
        </context-group>
      </trans-unit>
      <trans-unit id="54f29f9a6da150fc7c4fcd0b7e6d9a1b0314fd35" datatype="html">
        <source>Here are some links to help you start: </source>
        <target>당신이 시작하는데 도움이 될 만한 링크입니다. </target>
        <context-group purpose="location">
          <context context-type="sourcefile">app/app.component.html</context>
          <context context-type="linenumber">8</context>
        </context-group>
      </trans-unit>
      <trans-unit id="170b2bb80cfeeaf71c71cd4b56d240fdda4dfc0b" datatype="html">
        <source>Tour of Heroes</source>
        <target>영웅 여행</target>
        <context-group purpose="location">
          <context context-type="sourcefile">app/app.component.html</context>
          <context context-type="linenumber">12</context>
        </context-group>
      </trans-unit>
      <trans-unit id="4446b939821a1c94d99d8f88ebf5c67844d48d08" datatype="html">
        <source>CLI Documentation</source>
        <target>CLI 문서들</target>
        <context-group purpose="location">
          <context context-type="sourcefile">app/app.component.html</context>
          <context context-type="linenumber">17</context>
        </context-group>
      </trans-unit>
      <trans-unit id="f7b003c76057ba9ff6d99232971f826d015eaf54" datatype="html">
        <source>Angular blog</source>
        <target>앵귤러 블로그</target>
        <context-group purpose="location">
          <context context-type="sourcefile">app/app.component.html</context>
          <context context-type="linenumber">22</context>
        </context-group>
      </trans-unit>
    </body>
  </file>
</xliff>
```

angular.json 수정하자 

architect >> build >> configurations >> production을 복사해서 붙여넣기한후 이름을 ko로 바꾼다. 
```json
"ko": {
  "fileReplacements": [
    {
      "replace": "src/environments/environment.ts",
      "with": "src/environments/environment.prod.ts"
    }
  ],
  "optimization": true,
  "outputHashing": "all",
  "sourceMap": false,
  "extractCss": true,
  "namedChunks": false,
  "aot": true,
  "extractLicenses": true,
  "vendorChunk": false,
  "buildOptimizer": true,
  "budgets": [
    {
      "type": "initial",
      "maximumWarning": "2mb",
      "maximumError": "5mb"
    }
  ],
  //아래추가
  "outputPath": "dist/ko/browser",
  "i18nFile": "src/locale/message.ko.xlf",
  "i18nFormat": "xlf",
  "i18nLocale": "ko",
  "i18nMissingTranslation": "error"
}
```

architect >> serve >> configurations >> production을 복사해서 ko를 만든다.

```json
"production": {
  "browserTarget": "renderfarm-app:build:production"
},
"ko": {
  "browserTarget": "renderfarm-app:build:ko"
}
```

package.json에 start 다음에 추가
```json
"start": "ng serve",
"start:ko": "ng serve --configuration=ko",
```

실행 
```
npm run start:ko
```

http://localhost:4200/ 해보면 한글로 바귀어 나온다. 

![]({{site_baseurl}}/assets/angular-ssr-05.PNG)

i18n은 잘된것을 알수 있다. 이제 다른언어들도 다 변환하여 package.json에 추가하면 될 것이다.

### build ko (ssr없이)

package.json을 수정하여 build를 복사하여 build:ko를 만든다.
```
"build:ko": "ng build --configuration=ko",
```

npm run build:ko

### build ko (ssr)

package.json 에 추가하자.

```json
"build:ssr-ko": "npm run build:client-and-server-bundles-ko && npm run webpack:server-ko",
"build:client-and-server-bundles-ko": "ng build --prod --base-href=/ko --deploy-url=/ko/ --output-path=dist/ko/browser --i18n-file=src/locale/messages.ko.xlf --i18n-locale=ko --i18n-format=xlf  && ng run i18n-sample:server --outputPath=dist/ko/server",
"webpack:server-ko": "webpack --config webpack.server.config.js --env.lang=ko --progress --colors",
```

설명을 하면 빌드시 --base-href를 줘서 /ko가 기본경로가 되게 햇고 deploy-url을 이용해서 파일들 경로를 바꿔준다.  현재 앵귤러 6.0.8에 이부분에 조금 에러가 있는듯 싶다. html은 /ko를 붙여주지 않아서 /assets 처럼 /로 파일이 구성되 있다 나중에 처리할것이다.

빌드해보자.

npm run build:ssr-ko

결과는 다음과 같다. 

![]({{site_baseurl}}/assets/angular-ssr-08.PNG)

빌드된 아웃풋 경로를 기존에 dist/server에서 dist/ko/server로 바꾸려고 한다. 

webpack.server.config.js수정 

```js
const path = require("path");
const webpack = require("webpack");

module.exports = env => { //여기 수정
  let langWithDash = env == undefined ? "" : "-" + env.lang; //추가 
  let langWithSlash = env == undefined ? "" : "/" + env.lang; //추가
  return {
    entry: { server: "./server" + langWithDash + ".ts" },
    resolve: { extensions: [".js", ".ts"] },
    target: "node",
    mode: "none",
    // this makes sure we include node_modules and other 3rd party libraries
    externals: [/node_modules/],
    output: {
      path: path.join(__dirname, "dist" + langWithSlash),
      filename: "[name].js"
    },
    module: {
      rules: [{ test: /\.ts$/, loader: "ts-loader" }]
    },
    plugins: [
      // Temporary Fix for issue: https://github.com/angular/angular/issues/11580
      // for 'WARNING Critical dependency: the request of a dependency is an expression'
      new webpack.ContextReplacementPlugin(
        /(.+)?angular(\\|\/)core(.+)?/,
        path.join(__dirname, "src"), // location of your src
        {} // a map of your routes
      ),
      new webpack.ContextReplacementPlugin(/(.+)?express(\\|\/)(.+)?/, path.join(__dirname, "src"), {})
    ]
  };
};
```

언어를 env로 받아서 server-언어.ts로 해서 빌드를 하려고 하는중

기존 server.ts를 복사해서 server-ko.ts를 만들자. 
```ts
import "zone.js/dist/zone-node";
import "reflect-metadata";
import { enableProdMode } from "@angular/core";
import * as express from "express";
import { join } from "path";

enableProdMode();

const app = express();

const PORT = process.env.PORT || 4000;
const DIST_FOLDER = join(process.cwd(), "");

const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = require("./dist/ko/server/main"); //여기 수정

import { ngExpressEngine } from "@nguniversal/express-engine";
import { provideModuleMap } from "@nguniversal/module-map-ngfactory-loader";

app.engine(
  "html",
  ngExpressEngine({
    bootstrap: AppServerModuleNgFactory,
    providers: [provideModuleMap(LAZY_MODULE_MAP)]
  })
);
//이부분 추가 - 아래 설명
app.use(function(req, res, next) {
  if (req.url.slice(0, 4) === "/ko/") {
    req.url = req.url.slice(3);
  }
  next();
});

app.set("view engine", "html");
app.set("views", join(DIST_FOLDER, "browser"));

// Server static files from /browser
app.get("*.*", express.static(join(DIST_FOLDER, "browser")));

// All regular routes use the Universal engine
app.get("*", (req, res) => {
  console.log(`GET: ${req.originalUrl}`);
  res.render("index", {
    req: req,
    res: res
  });
});

// Start up the Node server
app.listen(PORT, () => {
  console.log(`Node server listening on http://localhost:${PORT}`);
});
```
위쪽에 읽어들어올 파일 경로 에 ko를 추가하였고  아래쪽에서 /ko/로 들어오는 트래픽에서 /ko 를 잘라내고 /로 바꿔서 다음으로 보낸다. 왜냐면 앱은 /로 들어올줄알고 있기 때문이다.



이제 dist아래 폴더를 서버에 올려보자. 

![]({{site_baseurl}}/assets/angular-ssr-08.PNG)

서버에서 web.config를 다음처럼 수정한다.

```xml
   
<rule name="All Traffic" stopProcessing="true">
  <match url=".*" />
  <action type="Rewrite" url="ko/server.js" /> //ko를 붙여줌
</rule>
```

접속해보면 한글로 나오는것을 알수 있다. 

![]({{site_baseurl}}/assets/angular-ssr-10.PNG)


## 한글 영어 동시 지원.

그럼 한글과 영어를 동시에 지원하려면 어떻게 해야할까?

빌드를 영어로 한번 한국어로 한번해서 같이 서버에 올리면될듯 싶다. 

해보자.

### 언어파일 추가 

messages.ko.xlf => messages.en.xlf

target을 영어로 만든다.

### server-en.ts를 만들자 기존걸 복사

필요한 부분을 수정해준다. ko 를 en으로 해주면됩니다.
```ts
...
const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = require("./dist/en/server/main"); //여기 수정

app.use(function(req, res, next) {
  if (req.url.slice(0, 4) === "/en/") { //여기 수정
    req.url = req.url.slice(3);
  }
  next();
});
...
```

### package.json에 추가하자.

```json
"build:ssr-en": "npm run build:client-and-server-bundles-en && npm run webpack:server-en",
"build:client-and-server-bundles-en": "ng build --prod --base-href=/en --deploy-url=/en/ --output-path=dist/en/browser --i18n-file=src/locale/messages.en.xlf --i18n-locale=en --i18n-format=xlf  && ng run i18n-sample:server --outputPath=dist/en/server",
"webpack:server-en": "webpack --config webpack.server.config.js --env.lang=en --progress --colors",
```

빌드해보자. 

npm run build:ssr-ko
npm run build:ssr-en

서버에 파일을 올리고 web.config를 수정한다. 기존 룰을 지우고 다음처럼 한다. 

```xml
<rule name="Lang-EN" stopProcessing="true">
    <match url="^en[\/].*?" />
    <action type="Rewrite" url="en/server.js" />
</rule>

<rule name="L51ang-KO" stopProcessing="true">
    <match url="^ko[\/].*" />
    <action type="Rewrite" url="ko/server.js" />
</rule>
```

![]({{site_baseurl}}/assets/angular-ssr-11.PNG)

이제 웹브라우저로 접속해본다. 

http://test.com/en/
http://test.com/ko/

한글 영어가 다 잘보인다. 

## 이제 /로 접속시 고객의 언어에 따라서 각 페이지로 보내보자. 

web.config를 다음을 추가한다.  
```xml
<rule name="Redirect To KO"  stopProcessing="true">
  <match url="^$" />
    <conditions>
      <add input="{HTTP_ACCEPT_LANGUAGE}" pattern="^ko" />
  </conditions>
  <action type="Redirect" url="{R:0}/ko/home" appendQueryString="true" />
</rule>

<rule name="Redirect To EN"  stopProcessing="true">
  <match url="^$" />
    <conditions>
      <add input="{HTTP_ACCEPT_LANGUAGE}" pattern="^en" />
  </conditions>
  <action type="Redirect" url="{R:0}/en/home" appendQueryString="true" />
</rule>

<rule name="Root Hit Redirect" stopProcessing="true">
    <match url="^$" />
    <action type="Redirect" url="/en/home" />
</rule>
```

브라우저 언어가 한글이면 /ko로 영어면 /en으로 둘다 아닌 다른거는 전부 /en으로 

## /assets 문제 

앵귤러 에러로 자꾸 /assets로 리퀘스트가 온다. 주로 이미지 파일이다. 이것도 리다이렉트로 처리해야한다. 

```xml
<rule name="Asset Redirect" stopProcessing="true">
  <match url="^assets\/.*" />
  <action type="Rewrite" url="dist/en/server.js" appendQueryString="true" />
</rule>
```

이러면 /assets로 온 리퀘스르를 en 으로 보내줘서 결과를 줄수가 있다. 

## 추가처리 

* http면 https로 보내기 
* 대소문자 섞여잇으면 전부 소문자로 변경하기 
* 400에러나 500에러가 나오면 /로 보내주기

이 내용은 깃허브에 올라가잇는 소스코드를 참고하기 바란다 전체 web.config는 다음과 같다. 

```xml
<configuration>
  <system.webServer>
    <httpErrors errorMode="Custom" defaultResponseMode="Redirect">
      <remove statusCode="404" />
      <remove statusCode="500" />
      <error statusCode="404" path="/en/home" responseMode="Redirect" />
      <error statusCode="500" path="/en/home" responseMode="Redirect" />
    </httpErrors>

    <!-- indicates that the hello.js file is a node.js application
    to be handled by the iisnode module -->
    <handlers>
      <!-- indicates that the app.js file is a node.js application to be handled by the iisnode module -->
      <add name="iisnode" path="server.js" verb="*" modules="iisnode" />
    </handlers>
    <rewrite>
      <rules>
        <!-- Don't interfere with requests for node-inspector debugging -->
        <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">
            <match url="^server.js\/debug[\/]?" />
        </rule>

        <rule name="Redirect to HTTPS" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
          <add input="{HTTPS}" pattern="^OFF$" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" appendQueryString="true" />
        </rule>

        <rule name="Convert to lower case" stopProcessing="true">
          <match url=".*[A-Z].*" ignoreCase="false" />
          <action type="Redirect" url="{ToLower:{R:0}}" redirectType="Permanent" />
        </rule>

        <rule name="Lang-EN" stopProcessing="true">
            <match url="^en[\/].*?" />
            <action type="Rewrite" url="dist/en/server.js" />
        </rule>

        <rule name="L51ang-KO" stopProcessing="true">
            <match url="^ko[\/].*" />
            <action type="Rewrite" url="dist/ko/server.js" />
        </rule>

        <rule name="Lang-JA" stopProcessing="true">
          <match url="^ja[\/].*" />
          <action type="Rewrite" url="dist/ja/server.js" />
        </rule>

        <rule name="Asset Redirect" stopProcessing="true">
          <match url="^assets\/.*" />
          <action type="Rewrite" url="dist/en/server.js" appendQueryString="true" />
        </rule>

        <rule name="Redirect To KO"  stopProcessing="true">
          <match url="^$" />
            <conditions>
              <add input="{HTTP_ACCEPT_LANGUAGE}" pattern="^ko" />
          </conditions>
          <action type="Redirect" url="{R:0}/ko/home" appendQueryString="true" />
        </rule>

        <rule name="Redirect To JA"  stopProcessing="true">
          <match url="^$" />
            <conditions>
              <add input="{HTTP_ACCEPT_LANGUAGE}" pattern="^ja" />
          </conditions>
          <action type="Redirect" url="{R:0}/ja/home" appendQueryString="true" />
        </rule>

        <rule name="Root Hit Redirect" stopProcessing="true">
            <match url="^$" />
            <action type="Redirect" url="/en/home" />
        </rule>

      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

## 개선해야할 곳 

* 언어 추가시마다  server.ts를 복사해서 경로를 변경해줘야하는데 조금 나은 방법이 있지 않을가?
* 한글로 보인다고 하더라도 소스보기를 하면 영어로 나온다. seo에는 좋지 않다.

## 소스코드 

<https://github.com/teamsmiley/i18n-sample>


## 추가 사항 

### 페이지에서 언어를 선택할수 있게 하기 

결과는 다음과 같다. 

![]({{site_baseurl}}/assets/angular-ssr-12.PNG)

app.components.ts 

```ts
import { Component, LOCALE_ID, Inject, OnInit } from "@angular/core";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"]
})
export class AppComponent {
  title = "app";

  currentLanguageCode: string = "en";

  languages = [
    { code: "en", label: "English", seleted: false },
    { code: "ko", label: "한국어", seleted: false },
    { code: "ja", label: "日本語", seleted: false }
  ];

  constructor(@Inject(LOCALE_ID) public localeId: string) {}

  ngOnInit() {
    if (this.localeId == "ko") this.currentLanguageCode = "ko";
    else if (this.localeId == "ja") this.currentLanguageCode = "ja";
  }
}

```

app.component.html
```html
<div class="mr-md-3">
  <select name="select" class="btn btn-outline-warning btn-sm" onchange="window.open(value,'_self');">
    <option *ngFor="let language of languages" value="/{{language.code}}/home" [selected]="language.code==currentLanguageCode">{{language.label}}</option>
  </select>
</div>
```

npm run build:ssr-ko
npm run build:ssr-en


