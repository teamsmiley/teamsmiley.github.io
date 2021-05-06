---
layout: post
title: 'chrome extension with angular'
author: teamsmiley
date: 2021-05-10
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# chrome extension with angular

```bash
npm install -g @angular/cli
cd ~/Desktop
ng new afk-chrome-extension > yes > yes > scss
cd afk-chrome-extension
npm i
ng build --prod
```

dist폴더를 봐보자. 그럼 컴파일되서 올라온것이 보인다.

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-33-31.png)

내용을 보려면

```bash
ng serve -o
```

이제 app.component.html 을 수정한다.

```html
<div>AFK</div>
<router-outlet></router-outlet>
```

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-37-12.png)

다음 같은 화면이 나오면 성공

이걸 이제 크롬에 등록해보자.

`chrome://extensions/`

developer mode를 켜고

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-38-04.png)

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-38-42.png)

load unpacked

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-39-22.png)

폴더를 선택해주면 된다.

잘 들어간것을 확인할수 있다.

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-41-48.png)

새창을 열면 잘 보인다.

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-42-31.png)

타이틀 값을 바꿔보자.

index.html을 수정하면된다.

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-43-29.png)

다시 빌드하고 크롬에서 리로드를 하면

![]({{ site_baseurl }}/assets/2021-05-09-chrome-extension-angular/2021-05-06-14-44-23.png)

title이 수정되었다.

<!-- ## backgroud script

background script

> the extension’s event handler; it contains listeners for browser events that are important to the extension.

```bash
npm i -D @types/chrome
```

vi tsconfig.app.json

```json
"types": [
  "chrome"
]
```

vi background.ts

```ts
chrome.runtime.onInstalled.addListener(() => {
  chrome.webNavigation.onCompleted.addListener(
    () => {
      chrome.tabs.query({ active: true, currentWindow: true }, ([{ id }]) => {
        chrome.pageAction.show(id);
      });
    },
    { url: [{ urlMatches: 'google.com' }] }
  );
});
```

```bash
npm i -D @angular-builders/custom-webpack
```

vi angular.json

```json
"projects": {
  "angular-chrome-extension": {
    ...
    "architect": {
      "build": {
        "builder": "@angular-builders/custom-webpack:browser",
        "options": {
          "customWebpackConfig": {
            "path": "./custom-webpack.config.js"
          }
          ...
        },
        ...
``` -->
