--- 
layout: post 
title: "jenkins github 연동" 
date: 2017-05-07 00:00  
author: teamsmiley 
tags: [CI]
image: /files/covers/blog.jpg
category: {CI}
---

# Windows 에서 Jenkins에서 github 가져오기 

별거 아닌거가지고 3일이나 삽질해서 스트레스 받음..

일단 정리는 해두자. 

작업전에 알아야할 것은 github에서는 외부에서 접속을 위해 ssh key를 사용한다. 

그런데  CI 서버에 그걸 넣어서 사용하다보면 CI 서버에서 코드를 수정해서 커밋도 할수 있게 된다.  

이것을 방지하기 위해 CI를 위해서  Deploy Key 를 제공한다. 이것은 코드를 가져만 올수 있다. 이걸 사용하자.

## jenkins server

* github desktop을 설치한다. 
* git shell도 같이 설치되는데 실행한다. 
* ssh key를 만들자.
```
ssh-keygen.exe -t rsa 
```
 
* C:\Users\Administrator\.ssh 폴더에  id_rsa id_rsa.pub 파일이 생성된다. (다른 유저를 사용중이면 C:\Users\\**UserName**\\.ssh)
* 젠킨스 루트 폴더에 .ssh폴더를 복사해서 붙여넣는다. C:\Program Files (x86)\Jenkins\.ssh 가 된다. 
* C:\Program Files (x86)\Jenkins\.ssh\id_rsa.pub를 메모장에서 열어서 내용을 복사한다. 

젠킨스 서버에서의 세팅은 완료되었음.

## github website 

* github에 로그인한다.
* github에 리파지토리 >> setting >> deploy keys

https://github.com/**{UserName}**/**{RepositoryName}**/settings/keys>

* 조금전 복사해둔 키를 추가한다. 


## jenkins 웹사이트 

* 젠킨스 웹화면에 들어간다. 
* 프로젝트를 추가한다. 
* source code management에서 git을 선택한다. 
* Repository URL : git@github.com:**{UserName}**/**{RepositoryName}**.git ( 콜론을 잘 보자. 리파지토리 이름뒤에도 .git이 붙는다. )

* credential을 추가한다. 
    * SSH Username with private key 
    * Private Key : from the Jenkins master ~./ssh 

이렇게 하고 빌드하면 젠킨스가  github에서 소스코드를 가져올수 있다. 
 
