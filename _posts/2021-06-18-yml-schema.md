---
layout: post
title: 'yml schema - vs code'
author: teamsmiley
date: 2021-06-18
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# yml schema - vs code

vs code에서 yml을 사용할때 github action 스키마를 이용해서 린트를 해보자.

## install

<https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml>

이 플러그인을 설치하자. yaml로 vscode에서 검색해도 된다.

## 설정

menu > code > preference > setting 에 아래 그림 부분을 클릭한다.

![]({{ site_baseurl }}/assets/2021-06-18-yml-schema/2021-06-17-07-05-27.png)

이제 설정을 추가한다.

```json
"yaml.schemas": {
    "https://json.schemastore.org/github-workflow.json":
      [
        ".github/workflows/*.yml",
        ".github/workflows/*.yaml"
      ]
},
```

설명을 조금 하자면 나의 경우에는 이상하게 github action yml파일이 스키마가 다른 파일로 잡혀서 에러가 나서 강제로 github action 스키마를 정해줬다.

쿠버네티스는 기본 지원이고 다른 스키마들도 적용할수 있다. glob 패턴을 사용해서 추가로 넣을수 있다.

다른 스키마를 찾아보려면 https://json.schemastore.org 에 가보면 엄청나게 많은 스키마를 이용할수 있는것을 알수 있다.

특별히 좋은 기능

- Auto completion : (Ctrl + Space) mac에서만 확인

옵션을 넣어야하는 곳에 커서를 둔다음에 ctl + space를 누르면 옵션 전체 내용을 확인할수 있다.

![]({{ site_baseurl }}/assets/2021-06-18-yml-schema/2021-06-17-07-12-33.png)

u 라고만 치면 u에 해당되는 선택할수 있는 옵션이 보여진다.

![]({{ site_baseurl }}/assets/2021-06-18-yml-schema/2021-06-17-07-09-34.png)

- 자동으로 포맷팅도 해준다.

기타는 위에 보이는 링크를 읽어보면 크게 의미가 있다.
