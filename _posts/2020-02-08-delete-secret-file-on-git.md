---
layout: post
title: 'Delete Secret File on Git' 
author: teamsmiley
date: 2020-02-08
tags: [git]
image: /files/covers/blog.jpg
category: {git}
---

# Delete Secret File On Git

git 에 커밋하지 말아야 할 파일을 커밋한 경우 `git rm filename`를 해도 히스토리에는 그대로 남아있게 된다. 

이 파일을 히스토리에서도 지워보자.

![]({{ site.baseurl }}/assets/2020-02-09-05-07-16.png

이 파일은 약 500메가나 되는 커밋하지 말아야할 파일이였다. 

## git filter-branch 사용
file
```bash
git rm adtree-20200205
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch adtree-20200205' \
--prune-empty  --tag-name-filter cat -- --all

git push --force
```

directory
```bash
git filter-branch --force --index-filter \
'git rm -r --cached --ignore-unmatch 폴더명' \
--prune-empty -- --all

git push --force
```
윈도우 cmd에서는 single qoute`'` 대신 double qoute`"`를 사용하고 한줄로 사용하시기 바랍니다.

## BFG Repo-Cleaner

https://rtyley.github.io/bfg-repo-cleaner/ 에서 jar를 다운로드 받는다.

java도 설치해야한다. 

```bash
git rm adtree-20200205
java -jar ~/Desktop/bfg-1.13.0.jar --delete-files adtree-20200205 . 
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force
```

