---
layout: post
title: 'Code Review - 3' 
author: teamsmiley 
date: 2018-05-14
tags: [code review]
image: /files/covers/blog.jpg
category: {program}
---

# code review - 3

시간 날 때마다 코드 리뷰를 해보고 문서를 작성하자. 

다음 컴포넌트를 보자.
```ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { combineLatest } from 'rxjs';
import { Category } from '../../models/category.model';
import { Post } from '../../models/post.model';
import { CategoryDataService } from '../../services/category-data.service';
import { PostDataService } from '../../services/post-data.service';

import * as showdown from 'showdown';

const converter = new showdown.Converter();

@Component({
  selector: 'www-post',
  templateUrl: './www-post.component.html',
  styleUrls: ['./www-post.component.css']
})

export class WwwPostComponent implements OnInit {
  boardName: string;
  categoryName: string;
  categories: Category[];
  category: Category;

  post: Post = new Post();
  markdownHtml: string;

  constructor(
    private route: ActivatedRoute,
    private postDataService: PostDataService,
    private categoryDataService: CategoryDataService) {
  }

  ngOnInit() {

    combineLatest([
      this.route.paramMap
    ])
      .subscribe(combined => {

        this.boardName = combined[0].get('boardname');
        this.post.id = combined[0].get('documentid');

        this.postDataService.getPost(this.boardName, this.post.id)
          .subscribe(result => {

            this.post = result;
            this.markdownHtml = converter.makeHtml(this.post.content);
            
            //이부분 이상함. 
            if (this.post.boardCategoryId != null) {
              this.categoryDataService.getCategory(this.boardName, this.post.boardCategoryId)
                .subscribe(result => {
                  this.category = result;
                });
            }
          });
      });
  }
}
```


