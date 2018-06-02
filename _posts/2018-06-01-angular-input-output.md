---
layout: post
title: 'Angular 6 컴포넌트 input output' 
author: teamsmiley 
date: 2018-06-01
tags: [angular]
image: /files/covers/blog.jpg
category: {program}
---

# 앵귤러에서 컴포넌트를 사용해 생각할 점을 적어본다.

## 현재 구조 

post 라는 컴포넌트에서  글제목 글내용 두개의 인풋박스를 사용하여 값을 서버로 보내는 중이다.

![]({{ site.baseurl }}/assets/component-1.PNG)

## 바뀌어야 할 구조 

글내용을 쓰는 곳을 에디터로 만들 계획이고 이것을 전체 프로젝트 이곳 저곳에서 사용할 계획이다 
editor를 컴포는트로 빼서  현재 페이지에서 사용해보자.

![]({{ site.baseurl }}/assets/component-2.PNG)

자식 컴포넌트를 만들고 

기존 컴포넌트에서 텍스트 박스를 자식 컴포넌트로 이동하고  부모 컴포넌트에서는 자식사용하는것으로 변경 

부모 
```html
<post-form-content-editor></post-form-content-editor>
```

자식 - 에디터
```html
<div>
    <span>Contents Write</span>
</div>
<textarea id="postformcontent"rows="6"> </textarea>
```

이제 컴포넌트를 두개를 사용하였으나 부모의 화면에 기존과 같이 보인다. 

## 글을 수정한다고 가정하고 페이지를 로딩하면 기존글이 보여야한다. 그렇다고 하면 기존글을 부모 컴포넌트는 가지고 있다. 이걸 자식 컴포넌트에 던져 줘야한다. 


자식 - 에디터 컴포넌트 
```ts
@Input() content: string = "";
```

인풋을 받을수 있고 텍스트박스에 이 인풋을 넣어야한다. 간단하게 ng model을 사용한다.

```html
<div class="section-divider">
  <span>Contents Write</span>
  </div>
  <textarea id="postformcontent" rows="6" [(ngModel)]="content"> </textarea>
</div>
```

이제 부모에서 값을 넣어줘야한다. 

부모 
```html
<post-form-content-editor [content]="post.content"></post-form-content-editor>
```

post.content가 부모가 서버에서 받아온 값이다. 이걸 content라는 태그를 사용해서 인풋을 시킨다.

이제 값이 화면에 보인다. 

## 참고 사항 - ngOnInit or ngOnChanges

ngOnInit함수에서 인풋받은 값을 찍어보자. 널이거나 안나온다. 

```ts
  ngOnInit() {
    console.log("ngOnInit:",this.content);
  }
  ngOnChanges() {
    console.log("ngOnChanges:",this.content);
  }
```

ngOnChanges후에 찍히는걸 알수가 있다. content를 마크다운으로 받아서 html로 변경을 하려고 하였는데 ngOnInit에서 아무리 변환을 해도 아무것도 안나와서 2시간 삽질함.

ngOnChanges후에 변환하니 잘됨. 

## 이제 텍스트박스 내용을 삭제하고 서버에 업데이트를 해야한다. 

텍스트박스를 업데이트를 하면 에디터 컴포넌트에서는 업데이트가 되엇으나 부모 컴포넌트에서는 이 값을 모른다. 

자식컴포넌트(에디터)가 부모 컴포넌트로 값이 변할때마다 현재값을 보내준면된다. 이벤트를 통해서 처리한다.

자식컴포넌트
```ts
@Output() contentChanged = new EventEmitter();
```

아웃풋을 이렇게 만들고 

```html
<textarea [(ngModel)]="content" (input)="onContentChanged()"> </textarea>
```

input이벤트가 일어나면 자기 자신 컴포넌트에 onContentChanged()함수를 호출한다. 아웃풋 contentChanged에 emit를 호출하면 이벤트가 발생이된다. 그 이벤트에 변경된 content가 담겨서 부모에게로 올라간다. 

```ts
onContentChanged() {
    this.contentChanged.emit(this.content);
}
```

부모는 일단 이걸 받아서 이용하자 여기서 event는 자식에서 보낸 글내용이다.
```ts
onParentContentChanged(event) {
  console.log("onContentChaged:", event);
  this.post.content = event;
}
```

```html
<post-form-content-editor (contentChanged)="onParentContentChanged($event)"></post-form-content-editor>
```

(contentChanged)  여기에서 contentChanged 와 자식 컴포넌트에서 아웃풋으로 설정한 것이다. 자식이 보내는 이벤트 이름이 contentChanged이고  부모는 그 이벤트를 잡아서 처리해야하므로 ()안에 이벤트 이름이 들어갔다.

```ts
@Output() contentChanged = new EventEmitter();
```
이것이 같아야한다.

알고 보면 별것아닌데 헷갈리네.

## 부모가 폼은 어떻게 만들어야할까? 
content를 지웠기 때문에 부모 폼에서는 빼야할까? 하는 고민이 있다. 코드를 보자.

```ts
  postFormGroup = new FormGroup({
    title: new FormControl('', Validators.required),
    content: new FormControl('', Validators.required),
    boardCategoryId: new FormControl(''),
  });
```
일단 content는 기존에도 있엇기 때문에 그대로 두고 자식 컴포넌트에서 값을 받을때 처리방법을 보자.

```ts
onContentChanged(event) {
  this.postFormGroup.patchValue({
    content: event
  });
}
```

폼그룹에 patchValue를 하면되는듯. validation은 잘되는지 궁금

## todo 

### 하나의 폼을 잘라 내면 유효성 검사는 어떻게 할것인가?

### 자식 컴포넌트에서 구지 ngmodel을 사용해야할가?





