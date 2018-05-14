---
layout: post
title: 'Code Review - 2 ' 
author: teamsmiley 
date: 2018-05-14
tags: [code review]
image: /files/covers/blog.jpg
category: {program}
---

# code review - 2

시간 날 때마다 코드 리뷰를 해보고 문서를 작성하자. 

```ts
uploadPostImg(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url, resource)
        .pipe(
            catchError(this.handleError)
        );

}

uploadPostThumbnail(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/thumbnail', resource)
        .pipe(
            catchError(this.handleError)
        );
}
```

## 함수 이름이 마음에 안든다. 
이 클래스는 파일을 업로드하는 기능인데 post라는 변수명이 갑자기 등장. 잘못된듯 싶고 img라는 이름은 꼭 이미지만 업로드하는 기능같아 보이는데 코드에는 그런 부분이 없다. 그냥 일반적인 파일업로드만으로 봐야해서 이름을 바꿔야겟다. 

```ts
uploadFile(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url, resource)
        .pipe(
            catchError(this.handleError)
        );

}

uploadThumbnailFile(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/thumbnail', resource)
        .pipe(
            catchError(this.handleError)
        );
}
```

## 코드에 중복을 제거 

코드를 보니까 중복이 많다.  매개변수로 빼면 금방 처리될듯 보인다.

```ts
uploadFile(addtionUrl: string, resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/' + addtionUrl, resource)
        .pipe(
            catchError(this.handleError)
        );
}

// uploadThumbnailFile(addtionUrl: string, resource: any): Observable<any> {
//     let url = `${this.apiUrl}boardfiles`;

//     return this.http.post(url + '/' + addtionUrl, resource)
//         .pipe(
//             catchError(this.handleError)
//         );
// }

//사용법
 this.fileDataService.uploadFile('', formData)
                .subscribe(result => {
                    console.log('uploaded');
                });
                
 this.fileDataService.uploadFile('thumnail', formData)
                .subscribe(result => {
                    console.log('uploaded');
                });
```

함수 하나로 바꿔져서 해결됬다.

### 사용하는곳에서 저장위치를 마음대로 바꿔버릴 염려가 있다. 

```ts
uploadFile(resource: any, isThumnail:boolean): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;
    if (isThumnail) url = url + '/thumnail';

    return this.http.post(url , resource)
        .pipe(
            catchError(this.handleError)
        );
}

//사용할때
this.fileDataService.uploadFile(formData,true)
                .subscribe(result => {
                    console.log('uploaded');
                });
```

이게 thumnail과 상위폴더 두개로 강제할수 있다. 


