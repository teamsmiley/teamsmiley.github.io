---
layout: post
title: "angular lint"
author: teamsmiley
date: 2020-09-08
tags: [angular]
image: /files/covers/blog.jpg
category: { program }
---

# ng lint

앵귤러 프로젝트를 만들면 package.json에 `ng lint` 가 들어있다. 사용해보자.

angular는 tslint를 사용하는데 현재는 eslint를 사용하게 추천을 하긴 하던데..일단 나는 tslint를 기본으로 제공해주니 사용해보기로 함.

## 현재 프로젝트에서 실행

`ng lint`

결과가...음...많이 에러라고 나온다. 하나씩 고쳐보자.

### vs lint

참고로 lint에 대한 설정은 프로젝트에 tslint.json에 전부 설정이 되잇으므로 수정할때 그 파일을 수정하면 되겟다.

lint결과를 보고 하나씩 수정을 해도 되겟지만 조금 편하게 하기 위하여 vscode에 tslint라는 플러그인을 설치해보자.

설치하고 나면 여러가지 경고를 볼수가 있다.

![]({{ site_baseurl }}/assets/2020-09-08-10-51-01.png)

설치하고 나면 이제 에러가 나는 부분에 코드들이 밑줄이 그어져서 보여진다. 이부분을 고치라는 이야기다.

고쳐보자.

![]({{ site_baseurl }}/assets/2020-09-08-10-52-41.png)

파일을 vscode에서 열어둔 상태에서 command + . 을 찍으면 다음 메세지가 나온다.

![]({{ site_baseurl }}/assets/2020-09-08-10-55-07.png)

auto fix를 선택을 하면 페이지에서 고쳐질수 잇는것들은 먼저 고쳐준다. 이것때문에 tslint를 설치해서 사용한다.

### 에러 메세지

- == should be ===

== 를 사용하지 말고 === 를 사용해라 (전부 수정)

- != should be !==

!= 를 사용하지 말고 !== 를 사용해라 (전부 수정)

- if statements must be braced (else statements must be braced )

if/else에 {} 괄호를 추가하자. (자동수정 가능)

- " should be '

" 때신 '를 사용하게 변경한다. (자동수정 가능)

- Identifier 'payload' is never reassigned; use 'const' instead of 'let'. ( prefer-const )

let을 사용하지 않고 const로 변경해준다.

- variable name must be in lowerCamelCase, PascalCase or UPPER_CASE

변수명에 \_ 로 시작하는것을 허용하지 않음 tslint.json에 다음 추가

```json
"variable-name": {
  "options": ["ban-keywords", "check-format", "allow-pascal-case", "allow-leading-underscore"]
},
```

- Exceeds maximum line length of 140

tslint.json

```json
"max-line-length": [true, 200],
```

- The selector should be prefixed by "app"

```json
"directive-selector": [true, "attribute", "camelCase"], // remove app
"component-selector": [true, "element", "kebab-case"], // remove app
```

- object access via string literals is disallowed

```json
"no-string-literal": false,
```

- Declaration of instance field not allowed after declaration of instance method. Instead, this should come at the beginning of the class/interface.

변수 선언후 함수가 나오면된다.

```ts
value: T;
disabled: boolean;
onChange(newVal: T) {}
onTouched(_?: any) {}
```

- for (... in ...) statements must be filtered with an if statement

## 기타 추천

```ts
if (value != null && value != undefined) {

==>

if (!!value) {
```

## git commit시에 lint

```bash
npm install husky --save-dev
```

vi package.json

```json
"scripts":{
...
},
"husky": {
  "hooks": {
    "pre-commit": "npm run lint && npm run build"
  }
},
```

```
git commit -am lint-init
```
