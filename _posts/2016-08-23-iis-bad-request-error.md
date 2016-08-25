---
layout: post
title: 'IIS Express - Bad Request Error - VS 2015' 
author: teamsmiley 
date: 2016-08-23
tags: [iis]
image: /files/covers/blog.jpg
category: {program}
---

# IIS Bad Request Error 

VS 2015를 사용중 기존에 테스트하던 http://localhost:5555 를 아이피로 접근 실패 

127.0.0.1:5555 도 접속이 안되고 
192.168.100.100:5555 도 접속이 안된다. 

여러군데를 검색해보앗지만 해결책이 정확하지 않아 삽질을 하다 해결했다. 

아주 간단햇다. 

```
C:\Users\{UserName}\Documents\IISExpress\config\applicationhost.config
```

또는 

```
ProjectFolder\.vs\applicationhost.config
```

두개의 파일을 확인해보자 . 둘중 하나에는 현재 사용하는 포트가 있을것이다. (나의 경우는 5555)
사용하는 포트로 검색해보면 다음같은 코드가 있을것이다. 

```xml
 <binding protocol="http" bindingInformation="*:50823:localhost" />
```

다음처럼 수정한다. 

```xml 
 <binding protocol="http" bindingInformation="*:50823:*" />
```

이제 접속이 잘 된다. 

VS를 재시작후 다시 확인해보면 설정파일에 설정이 하나 더 생겻음을 알수 있다.
계속 생긴다...

원인은 localhost가 없으면 계속 만드는 듯 싶다. 
그래서 꼭 아래처럼 2줄을 해줘야 새로 만들지 않는다.


```xml
 <binding protocol="http" bindingInformation="*:50823:localhost" />
 <binding protocol="http" bindingInformation="*:50823:*" />
```

* 참고 사항 

방화벽때문에 접속이 안될수도 있다 꼭 방화벽 세팅을 체크하기 바란다. 



