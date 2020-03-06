---
layout: post
title: 'Angular Interceptor' 
author: teamsmiley
date: 2020-03-06
tags: [angular]
image: /files/covers/blog.jpg
category: {angular}
---

# Angular Interceptor 사용중 header 처리

오늘 생긴일인데 기존처럼 httpclient로 post데이터를 날리는데 에러가 나기 시작했다.

request를 확인해보니 content-type이 text/plain으로 되있다. 분명 application/json으로 넣어둿는데. 뭐가 문제일가?

```ts
create(resource: any) {
  return this.http
    .post(this.url, JSON.stringify(resource), {
      headers: { 'Content-Type': 'application/json' }
    })
    .pipe(catchError(this.handleError));
}
```

분명 보내고 있다. 그런데 얼마전에 interceptor를 바꾼 기억이 나서 확인해보았다. 

Interceptor코드는 다음과 같다. 

```ts
if (req.url.startsWith(environment.apiUrl)) {
  return from(this._authService.getAccessToken().then(token => { //token을 가져와서 
    const headers = new HttpHeaders().set('Authorization', `Bearer ${token}`); //header에 세팅을 한다.

    const authReq = req.clone({ headers }); //header는 immutable이므로 수정이 되지 않아서 복사를 해서 하나더 만들어서 넣어준다.
    return next.handle(authReq).pipe(tap(_ => { }, error => {
      var respError = error as HttpErrorResponse;
      if (respError && (respError.status === 401 || respError.status === 403)) {
        this._router.navigate(['/unauthorized']);
      }
    })).toPromise();
  }));
}
else {
  return next.handle(req);
}
```

헤더에서 token을 추가하는 과정에서 기존걸 다 버리고 authorize token만 넣어 준다. 

기존 헤더를 다 가지고 오고 추가로 authorize token을 넣어줘야 될거같다.

해보자. 다음과 같이 수정하였다.

```ts
if (req.url.startsWith(environment.apiUrl)) {
  return from(this._authService.getAccessToken().then(token => {

    const headerSettings: { [name: string]: string | string[]; } = {}; // header setting을 담아둘 배열을 생성 

    for (const key of req.headers.keys()) {  //기존 헤더에서 키값과 벨류값을 가져와서 배열에 저장 
      headerSettings[key] = req.headers.getAll(key);
    }

    if (token) {
      headerSettings['Authorization'] = 'Bearer ' + token; // 토큰이 잇으면 authorize token을 헤더에 추가한다. 
    }

    const newHeaders = new HttpHeaders(headerSettings); //token setting으로 header를 만든다. 

    const authReq = req.clone({ headers: newHeaders }); //header는 immutable이므로 수정이 되지 않아서 복사를 해서 하나더 만들어서 넣어준다.
```

이렇게 하면 기존 헤더를 가져와서 조작후 새 헤더를 만들어서 request에 추가해준다. 

잘 해결됬다.



