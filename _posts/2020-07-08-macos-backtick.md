---
layout: post
title: 'macos - backtick' 
author: teamsmiley
date: 2020-07-08
tags: [macos]
image: /files/covers/blog.jpg
category: {macos}
---

# macos에서 '₩' 대신 '`' 입력하기

macos에서 영어일때는 `이 입력이 잘되는데 한글일때는 '₩' 가 입력이 되서 많이 불편하다.

특히 마크다운에서 코드를 적을때 ```을 사용해야 해서 많이 불편하다.

한글에서도 '`'를 입력하게 바꿔보자.

```
mkdir -p ~/Library/KeyBindings/
vi ~/Library/KeyBindings/DefaultkeyBinding.dict
```

```
{
  "₩" = ("insertText:", "`");
}
```

DefaultkeyBinding.dict 파일을 만들고 위 내용을 저장후 재부팅하면 한글에서도 backtick을 사용할수 있다.



