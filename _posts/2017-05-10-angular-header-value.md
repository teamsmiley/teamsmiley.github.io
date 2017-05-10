--- 
layout: post 
title: "angular 2 pagination" 
date: 2017-05-10 00:00  
author: teamsmiley 
tags: [Angular]
image: /files/covers/blog.jpg
category: {Angular}
---

# angular 2 pagination  

## 기본 코드 작성 
ng g component pagination

코드를 가져와서 적당히 맞춘다음에 

```html
<nav *ngIf="totalItems > pageSize">
        <ul class="pagination">
            <li [class.disabled]="currentPage == 1">
                <a (click)="previous()" aria-label="Previous">
                <span aria-hidden="true">&laquo;</span>
                </a>
            </li>
            <li [class.active]="currentPage == pageNumber" *ngFor="let pageNumber of pages" (click)="changePage(pageNumber)">
                <a>{{ pageNumber }}</a>
            </li>
            <li [class.disabled]="currentPage == pages.length">
                <a (click)="next()" aria-label="Next">
                <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
        </ul>
</nav>   
```

```ts
import { Component, Input, 	Output, EventEmitter,OnChanges   } from '@angular/core';

@Component({
  selector: 'pagination',
  templateUrl: './pagination.component.html',
  styleUrls: ['./pagination.component.css']
})
export class PaginationComponent implements OnChanges {
  	@Input('total-items') totalItems;
	@Input('page-size') pageSize = 10;
	@Output('page-changed') pageChanged = new EventEmitter();
	pages: any[];
	currentPage = 1; 

	ngOnChanges(){
    this.currentPage = 1;
        
		var pagesCount = Math.ceil(this.totalItems / this.pageSize); 
		this.pages = [];
		for (var i = 1; i <= pagesCount; i++)
			this.pages.push(i);
	}

	changePage(pageNumber){
		this.currentPage = pageNumber; 
		this.pageChanged.emit(pageNumber);
	}

	previous(){
		if (this.currentPage == 1)
			return;

		this.currentPage--;
		this.pageChanged.emit(this.currentPage);
	}

	next(){
		if (this.currentPage == this.pages.length)
			return; 
		
		this.currentPage++;
		this.pageChanged.emit(this.currentPage);
	}
}
```
이제 사용하자. 

html에서 사용할때 일단 total-item은 하드코딩..나중에 해결하자.
```html
<pagination [total-items]="10" [page-size]="query.pageSize" [items]="courses" (page-changed)="onPageChanged($event)"></pagination>
```
ts에서  구현해주자.
```ts
export class CourseListComponent implements OnInit {
    private readonly PAGE_SIZE = 3;

    query : any ={
        pageNumber:1, 
        pageSize:this.PAGE_SIZE
    };
    ...

    onPageChanged(pageNumber) {
        this.query.pageNumber = pageNumber;
        this.populateCourses();
	}
```    

### 서비스에서 쿼리스트링을 만들어서 던져야 한다. 

```ts
export class CourseService {
    ...
     getCourses(query) {
        return this._http.get( this._url +'?'+this.toQueryString(query))
            .map(res => res.json());
    }

    toQueryString(obj) {
        var parts = [];
        for (var property in obj) {
            var value = obj[property];
            if (value != null && value != undefined) 
                parts.push(encodeURIComponent(property) + '=' + encodeURIComponent(value));
        }

        return parts.join('&');
    }
    ...
```
페이지를 확인하면 잘 나온다... 문제는 전체 페이지 갯수가 안맞다. 

아래처럼 해결하자.

## header에  pagination을 받아서 처리 하는 방법 (total-item을 서버에서 받아서 넣어줌.)

### 기존에 리스트만 리턴하던 코드를 total Count도 함께 리턴하여 사용 할수 있음.

기존 코드 
```ts
getCourses(query) {
        return this._http.get( this._url +'?'+this.toQueryString(query))
             .map((res :Response) => res.json())
             .catch((error:any) => Observable.throw(error || 'Server error'))
             ;
}
```
위와 같이 일반 형태이다. 

그런데 totalCount를 헤더값으로 받고 있다.  형태는 아래와 같다. 

```
X-Pagination:{
    "totalCount":4,
    "pageSize":3,
    "currentPage":1,
    "totalPages":2,
    "previousPageLink":null,
    "nextPageLink":"http://localhost:5000/api/courses?orderBy=Id&pageNumber=2&pageSize=3"
}
```

현재는 totalcount만 가져와서 리턴해주고 싶다. 

//수정된 코드 

```ts
getCourses(query) { // 리턴값 타입을 선언해준다.꼭 필요한건 아님
        return this._http.get( this._url +'?'+this.toQueryString(query))
             .map((res :Response) => {
                 const totalCount = + JSON.parse(res.headers.get("X-Pagination")).totalCount;
                 let results = res.json();
                 
                 return {
                     items : results,
                     totalCount:totalCount
                 }
                })
             .catch((error:any) => Observable.throw(error || 'Server error'))
             ;
}
```


이렇게 리턴해주면 받는쪽에서 다음처럼 사용하면된다. 
```ts
this._courseService.getCourses(this.query).subscribe(
            response => { 
                this.courses = response.items;
                this.totalCount = response.totalCount;
            }
     );
```

## pagination을 body로 넘기는 방법 API 수정이 필요( 이게 이해는 쉽지만 추천하지는 않는다. rest 원칙에 맞지않음.)

resource list와 total count를 가져오는법 

일단 return할 Resource객체를 만들자. 
```cs
public class QueryResultResource<T>
{
    public int TotalItems { get; set; }
    public IEnumerable<T> Items { get; set; }
}
```

기존 컨트롤러를 다음처럼 바꾸자. 
QueryResultResource를 만들고 items와 totalCount를 넣어준다.그걸 리턴한다. 
```cs
...
//var result = Mapper.Map<IEnumerable<Course>, IEnumerable<CourseResource>>(entities);
//return Ok(result.ShapeData(param.Fields));
...
var result = new QueryResultResource<Course>(); 
result.Items = entities;
result.TotalItems = entities.TotalCount;
return Ok(result);
```

기존에 리스트가 아이템으로 묵이고 전체 갯수가 나온다.
```json
// 기존 
[
  {
    "id": "f426d7bb-253a-4d2d-3134-08d49414f243",
    "name": "angular",
    "description": "anuglar",
    "level": 0,
    "fullPrice": 0
  },
  {
    "id": "291f8bc8-b361-46fd-5520-08d4969ea202",
    "name": "asdFASDF",
    "description": "asdf",
    "level": 0,
    "fullPrice": 0
  }
]
// 새로운 방식 
{
  "totalItems": 4,
  "items": [
    {
      "id": "f426d7bb-253a-4d2d-3134-08d49414f243",
      "name": "angular",
      "subTitle": null,
      "description": "anuglar",
      "level": 0,
      "fullPrice": 0,
      "createDateTime": "2017-05-05T17:15:05.0673107",
      "sections": []
    },
    {
      "id": "291f8bc8-b361-46fd-5520-08d4969ea202",
      "name": "asdFASDF",
      "subTitle": null,
      "description": "asdf",
      "level": 0,
      "fullPrice": 0,
      "createDateTime": "2017-05-08T22:45:43.2972827",
      "sections": []
    }
  ]
}
```

이렇게 처리할수도 있지만 헤더에 넘기는것을 추천함. 

