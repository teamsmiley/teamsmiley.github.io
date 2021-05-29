---
layout: post
title: 'vscode-formatter'
author: teamsmiley
date: 2021-05-28
tags: [devops]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# vscode tip

언어별로 포멧터를 다르게 설정할수도 있다.

## ts/js formatter

js나 ts의 경우에는 Prettier를 사용중이고

https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

이곳에서 관련 내용을 확인할수 있다.

## c# formatter

![]({{ site_baseurl }}/assets/2021-05-28-vscode-formatter/2021-05-28-19-57-19.png)

setting에서 확인해보기 바란다.

자세한 옵션은 여기서 볼수 있다.

https://docs.microsoft.com/en-us/dotnet/api/microsoft.codeanalysis.csharp.formatting.csharpformattingoptions?view=roslyn-dotnet

잘되는지는 모르겟음. 기본 포멧터가 잘되는듯하여 구지 신경쓰지않음.

## 현재 setting

```json
{
  "editor.formatOnPaste": true,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": null,

  "prettier.printWidth": 200,
  "prettier.singleQuote": true,

  "omnisharp.organizeImportsOnFormat": true,
  //언어별로 설정이 가능
  "[markdown]": {},
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
    // "editor.quickSuggestions": true
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## 코드 2줄 이상이 빈줄을 1줄로

`&#8984; + F`

![]({{ site_baseurl }}/assets/2021-05-28-vscode-formatter/2021-05-28-20-08-06.png)

```txt
^[\r\n]{2,}
\r\n
```

또는 `/r`을 빼고 해보셔도 된다.
