---
layout: post
title: "angular bundle size"
author: teamsmiley
date: 2020-09-07
tags: [angular]
image: /files/covers/blog.jpg
category: { program }
---

# angular bundle size 확인

## 현재 상태 확인

```bash
npm install -g webpack-bundle-analyzer

ng build --stats-json # generate ./dist/stats.json
ng build --stats-json --prod # generate ./dist/stats.json

webpack-bundle-analyzer ./dist/stats.json
```

웹서버가 실행되면서 분석한 내용을 그림으로 보여줌.

### material 파일중 사용하는것만 임포트하게 수정하면 다음처럼 변경이 됨.

## 변경전

![]({{ site_baseurl }}/assets/2020-09-07-17-07-53.png)

![]({{ site_baseurl }}/assets/2020-09-08-00-27-00.png)

## 변경후

![]({{ site_baseurl }}/assets/2020-09-08-00-21-44.png)

![]({{ site_baseurl }}/assets/2020-09-08-00-29-04.png)

## 결론

1메가 정도 용량이 준것을 알수 있다. 사용하는 패키지만 적용되었다.

```ts
import { MatCardModule } from "@angular/material";
```

이런식으로 사용하면 전체를 다 로딩해버리기 때문에 꼭

```ts
import { MatCardModule } from "@angular/material/card";
```

이렇게 사용해야한다.

## 프로덕션 모드로 빌드후 결과

![]({{ site_baseurl }}/assets/2020-09-08-00-54-18.png)
