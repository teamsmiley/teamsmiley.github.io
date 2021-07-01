---
layout: post
title: 'npm shrinkwrap'
author: teamsmiley
date: 2021-07-01
tags: [code]
image: /files/covers/blog.jpg
category: { program }
---

# npm shrinkwrap

잘되던 프로젝트가 build가 안되는 문제가 발생햇다.

왜 그런지 확인을 해보니 capacitor가 2에서 3으로 업데이트되면서 뭔가가 바뀐것 같다.

일단 뭐가 바뀌엇는지는 따로해결하기로 하고 빌드부터 해결하기로 햇다.

원인을 찾아보니 ionic-appauth 패키지가 capacitor-secure-storage-plugin 을 사용한다.

현재는 0.4.0을 사용하는데 이 버전을 0.5.1로 해주면 해결이 된다.

문제는 ionic-appauth 에서 관리되는 버전을 어떻게 바꾸는지가 문제가 됬다.

검색을 해보니 npm shrinkwrap 가 보인다.

일단 프로젝트에 npm-shrinkwrap.json 파일을 만들고 다음처럼 작성했다.

```json
{
  "dependencies": {
    "capacitor-secure-storage-plugin": {
      "version": "0.5.1",
      "dependencies": {
        "@capacitor/core": {
          "version": "2.4.7"
        }
      }
    }
  }
}
```

`ioinc build`를 해보면 문제없이 잘 되는것을 알수있다.

일단 해결 완료.

## vscode에서 워닝

![]({{ site_baseurl }}/assets/2021-07-01-npm-shrinkwrap/2021-07-01-07-49-07.png)

![]({{ site_baseurl }}/assets/2021-07-01-npm-shrinkwrap/2021-07-01-07-49-44.png)

5000 -> 50000으로 일단 변경해두고 사용하자.
