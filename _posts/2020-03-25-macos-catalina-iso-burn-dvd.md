---
layout: post
title: 'burn iso to dvd with macos catalina' 
author: teamsmiley
date: 2020-03-25
tags: [visual studio]
image: /files/covers/blog.jpg
category: {macos}
---

# macos catalina에서 dvd 만들기 

iso파일을 가지고 dvd를 만들어야하는데 catalina에서 버그가 잇어서 자꾸만 다음처럼 에러가 나온다.

```
ISO file is invalid
```

검색을 해보니 카타리나에서 버그가 있어서 안되는듯 보인다.

다행이 터미널에서 처리할수 있다고 한 내용이 있어서 여기에 적어둔다.

터미널을 켜고

```bash
hdiutil burn ~/Desktop/DiskImageFile.iso
```

이렇게 하면 cdrom을 찾아서 시디를 구워준다.


