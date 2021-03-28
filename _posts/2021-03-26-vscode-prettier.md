---
layout: post
title: 'Prettier-Code Formatter'
author: teamsmiley
date: 2021-03-26
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# Prettier-Code Formatter

## Install

CMD+P \(Ctrl+P\)

```bash
ext install esbenp.prettier-vscode
```

설정값 우선순위

```text
settings.json > .editorconfig > .prettierrc
```

## 기본 설정

Default Formatter에 설정하면 모든 파일에 적용이된다.

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

원하는 언어에만 적용해도 된다.

```json
{
  "editor.defaultFormatter": null,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## 추가 옵션

- prettier.arrowParens \(default: 'avoid'\) " : arrow 함수에 매개변수에 괄호를 붙임
- prettier.bracketSpacing \(default: true\)
- prettier.endOfLine \(default: 'auto'\)
- prettier.htmlWhitespaceSensitivity \(default: 'css'\)
- prettier.jsxBracketSameLine \(default: false\)
- prettier.jsxSingleQuote \(default: false\) : jsx에서 single qoute를 사용
- prettier.printWidth \(default: 80\)
- prettier.proseWrap \(default: 'preserve'\)
- prettier.semi \(default: true\) : 문장뒤에 세미콜론을 붙일지 뺄지 설정
- prettier.singleQuote \(default: false\) : single qoute를 사용
- prettier.tabWidth \(default: 2\)
- prettier.trailingComma \(default: 'none'\)
  - none - 쉼표없음
  - es5 - ES5에서 유효한 후행 쉼표 붙힘
  - all - 모두 후행 쉼표를 붙힘
- prettier.useTabs \(default: false\) : 탭이 있는 줄을 들여 쓰기 합니다.
- prettier.requireConfig \(default: false\)
- prettier.ignorePath \(default: .prettierignore\) : `.prettierignore`에 파일명을 기록하면 그 파일은 무시

## 나의 설정

일단 markdown에서는 prettier가 적용되면 `_` 가 `\_` 되는 경우가 많고 `(`도 escape가 되는경우가 `\(` 있어서 사용하지 않고 있다.

붙여넣기나 저장시 자동 formatting을 하도록 해두었다. width는 200을 사용한다.

```json
"editor.formatOnPaste": true,
"editor.formatOnSave": true,

"[markdown]": {

},
"[html]": {
  "editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[typescript]": {
  "editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[json]": {
  "editor.defaultFormatter": "esbenp.prettier-vscode"
},
"prettier.printWidth": 200,
```

## 문제

오늘 작업을 하는데 마크다운에서 format을 하면 `_` 가 `\_` 로 변경이 된다. 분명 설정에서

```json
"editor.defaultFormatter": null,
"[markdown]": {

},
```

이렇게 마크다운은 prettier를 사용하지않게 해두었는데도 실행이 된다.

```json
prettier.disableLanguages: [
"markdown"
]
```

이 기능은 depreciate가 됬다. `.prettierignore`를 사용하라고 한다. 파일 생성후 다음처럼 작성하니 드디어 prettier의 동작이 md파일에는 적용이 안된다.

```.gitignore
**/*.md
```

왜 처음 세팅이 안되는지는 아직도 모르겟다.
