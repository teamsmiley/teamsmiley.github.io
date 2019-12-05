---
layout: post
title: 'Angular DatePicker with Momentjs' 
author: teamsmiley
date: 2019-12-04
tags: [angular]
image: /files/covers/blog.jpg
category: {angular}
---

# angular datepicker with momentjs

자바스크립트 date를 다루기가 참 불편했다. 이걸 한방에 해결해주는 momentjs를 사용해봤다. 

material date picker를 사용하면 날짜를 선택해서 서버로 보내면 브라우저의 타임존을 같이 보내준다. 한국고객과 미국 서버의 경우 하루가 차이가 나게 된다. 분명 10일을 선택햇는데 디비에는 9일로 저장이 된다. 한국 시간을 pst로 바꿔서 저장을 하다보니 그런듯 보임

이걸 해결하기 위해 고민해 보았다.
1. 타임존을 고려해서 디비에 저장후 브라우저에서 다시 바꿔서 보여주기 
1. 타임존을 무시하고 날짜를 브라우저에서 짤라서 yyyy/MM/DD만 보내주기 서버에서는 이걸로 date 를 만들어서 처리하기 

프로젝트가 급해서 일단 되는데로 했다.  타임존을 무시하고 yyyy/MM/DD만 서버로 보내기로함. 

momentjs를 써서 date포맷을 편하게 할수 있다.


install npm

```
npm install moment --save 
npm install @angular/material-moment-adapter --save
```

material.module.ts
```ts
import { MatMomentDateModule, MomentDateAdapter, MAT_MOMENT_DATE_FORMATS } from '@angular/material-moment-adapter';

exports [
    MatMomentDateModule,
]
  
providers: [
    { provide: DateAdapter, useClass: MomentDateAdapter, deps: [MAT_DATE_LOCALE] },
    { provide: MAT_DATE_FORMATS, useValue: MAT_MOMENT_DATE_FORMATS }
  ]
```

설정 완료 

이제 date를 사용하는곳에서 에서 다음처럼 처리 
```ts
import * as moment from 'moment';

this.visaFrom.setValue(moment(this.visaFrom.value).format('YYYY/MM/DD'));
```
설명을 하면 폼에서 받은 값을 yyyy/MM/DD로 바꿔서 다시 셋을 하고 서버로 넘겨준다. 

그럼 이제 서버로 yyyy/mm/dd가 넘어가는것을 알수 있다.

## error

* this.visaFrom.value 값이 null로 서브밋이 되야할때 에러가 남.

```ts
import * as moment from 'moment';

this.visaFrom.setValue(this.visaFrom.value != undefined ? moment(this.visaFrom.value).format('YYYY/MM/DD') : null);
```

* new Date() 코드는 다음처럼 변경해서 사용하면 편하다. 
```ts
//new Date()
moment(new Date()).format('YYYY/MM/DD')
```

