---
layout: post
title: 'AutoMapper-TS' 
author: teamsmiley 
date: 2018-05-11
tags: [automapper]
image: /files/covers/blog.jpg
category: {program}
---

# Automapper ts 사용하기 

c#으로 automapper를 써서 코드를 간결하게 만들어서 잘쓰고 있는데 앵귤러 프로젝트를 하는데 모델들 관리가 힘들어서 automapper 비슷한걸 찾아보다 이걸 찾았다. 

사용해보니 코드가 간결해지고 명확해져서 너무 좋다.


## install

npm install automapper-ts --save

## angular.json수정 

scripts에 추가해 주면 된다.
```json
"styles": [
    "styles.css"
],
"scripts": [
    "./node_modules/automapper-ts/dist/automapper.min.js"
],
```

## app.component.ts 수정 
이걸 추가해준다
```ts
import {  } from "automapper-ts";
```

## model을 만들자.

```
cd c:\Users\ragon\Desktop\Github\rc4_console\console\src\app\board\models

ng g class Post --type=model

```

```ts
export class Post{
    id: string ="";
    title: string = "";
    content: string = "";
    regDate: Date = new Date();
    boardCategoryId: string = null;
    isPublish: boolean = false;
    thumbnailFullPath : string =  "";
}

export class PostAdmin extends Post{
    publish: boolean;
}

export abstract class PostForManipulationAbstract {
    Title:string;
    RegDate: Date ;
    Content: string ;
    BoardCategoryId:string;
}

export class PostForCreation extends PostForManipulationAbstract{
    IsPublish: boolean = false;
}

export class PostForUpdate extends PostForManipulationAbstract{
    IsPublish: boolean;
    ThumbnailFullPath: string;
}
export class PostAdminForCreation extends PostForCreation {
}

export class PostAdminForUpdate extends PostForUpdate{
}
```

## 매핑 설정 

appmodule.ts에 construct에 다음을 추가한다. 

```ts
    automapper.createMap('PostFormGroup', 'PostForCreation')
      .forSourceMember('id', (opts: AutoMapperJs.ISourceMemberConfigurationOptions) => { opts.ignore(); })
      // .forMember('bandid', function (opts) { opts.mapFrom('band'); })
      ;
      automapper.createMap('PostFormGroup', 'PostAdminForCreation')
      .forSourceMember('id', (opts: AutoMapperJs.ISourceMemberConfigurationOptions) => { opts.ignore(); })
      .forMember('isPublish', function (opts) { opts.mapFrom('isPublish'); })
      ;
```

폼그룹을 create하고 매핑하는데 id는 무시를 해라. 

어드민의 경우 id는 무시하고 isPublish를 추가로 넣어서 던져라.

## 매핑 사용
```ts
create(): void {
    if (this.postFormGroup.dirty) { //form create은 항상 체크해줄것.
      if (this.isAdmin === true) {
        
        // create postWithAdminForCreation from form model
        let post = automapper.map(
          'PostFormGroup',
          'PostAdminForCreation',
          this.postFormGroup.value);

        this.postDataService.createPost(this.boardName, this.postFormGroup.value)
          .subscribe(result => {
            this.router.navigate(['/boards', this.boardName, 'posts']);
          }, error => {
            console.log(error);
          });
      }else {
         // create postWithAdminForCreation from form model
         let post = automapper.map(
          'PostFormGroup',
          'PostForCreation',
          this.postFormGroup.value);

        this.postDataService.createPost(this.boardName, this.postFormGroup.value)
          .subscribe(result => {
            this.router.navigate(['/boards', this.boardName, 'posts']);
          }, error => {
            console.log(error);
          });
      }
    }
  }
```



