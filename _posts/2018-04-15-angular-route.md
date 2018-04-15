# angular route 가져오기

## url을 파싱해서 값을 가져오기 

* url 대소문자를 구분한다.(나중에 이것도 처리해야함)

## component가 자기자신에게 돌아오지 않는경우

```ts
let pagesize = this.route.snapshot.paramMap.get('pagesize');
let pagesize = this.route.snapshot.queryParamMap.get('pagesize');
```

## 컴포넌트가 자기자신에게 돌아올 때

```ts
this.route.queryParamMap
  .subscribe(params => {

    let pageSize = params.get('pagesize'); //'pagesize'가 다 소문자여야함.
    this.query.pageSize = (pageSize == null) ? this.query.pageSize : +pageSize;
    console.log(pageSize);

    let currentPage = params.get('currentpage');//'currentpage'가 다 소문자여야함.
    this.query.currentPage = (currentPage == null) ? this.query.currentPage : +currentPage;
    console.log(currentPage);

    this.postsService.getAll("documents", this.query)
      .subscribe(result => {
        this.posts = result.payload;
        this.pagination = JSON.parse(result.headers.get('x-pagination'));
        console.log(this.pagination);
      }, error => {
        console.log(error);
      });
  });
```

## 두개의 route를 subscribe할때

```ts
Observable.combineLatest([
  this.route.paramMap,
  this.route.queryParamMap
])
  .subscribe(combined => {
    this.boardId = combined[0].getAll('boardId');

    let pageSize = combined[0].get('pagesize');
    this.query.pageSize = (pageSize == null) ? this.query.pageSize : +pageSize;

    let currentPage = combined[0].get('currentpage');
    this.query.currentPage = (currentPage == null) ? this.query.currentPage : +currentPage;

    this.postService.getAll("documents", this.query)
      .subscribe(result => {
        this.posts = result.payload;
        this.pagination = JSON.parse(result.headers.get('x-pagination'));
        console.log(this.pagination);
      }, error => {
        console.log(error);
      });
  });
```