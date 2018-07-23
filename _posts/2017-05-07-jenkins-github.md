--- 
layout: post 
title: "jenkins github 연동" 
date: 2017-05-07 00:00  
author: teamsmiley 
tags: [CI]
image: /files/covers/blog.jpg
category: {CI}
---

# Jenkins에서 Github 연동 - Windows

## repository 가져오기

별거 아닌거가지고 3일이나 삽질해서 스트레스 받음..

일단 정리는 해두자. 

작업전에 알아야할 것은 github에서는 외부에서 접속을 위해 ssh key를 사용한다. 

그런데  CI 서버에 그걸 넣어서 사용하다보면 CI 서버에서 코드를 수정해서 커밋도 할수 있게 된다.  

이것을 방지하기 위해 CI를 위해서  Deploy Key 를 제공한다. 이것은 코드를 가져만 올수 있다. 이걸 사용하자.

## jenkins server

* github desktop을 설치한다. 
* git-bash 실행한다. (C:\Program Files\Git)
* ssh key를 만들자.
```
ssh-keygen.exe -t rsa 
```
 
* C:\Users\Administrator\.ssh 폴더에  id_rsa id_rsa.pub 파일이 생성된다. (다른 유저를 사용중이면 C:\Users\\**UserName**\\.ssh)

* C:\Users\Administrator\.ssh\id_rsa.pub를 메모장에서 열어서 내용을 복사한다. ==>이건 github에 설정 

* id_rsa 는 젠킨스에 나중에 설정


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

* credential을 추가한다. ADD >> jenkins >> 
    * SSH Username with private key 
    * Username : Administrator
    * Private Key : Enter Directly id_rsa파일을 붙여 넣는다.
    * Passphrase : 인증서 만들때 넣은값이다. 일반적으로 아무것도 안넣는다.


![]({{site_baseurl}}/assets/jenkins-github-01.png)


이렇게 하고 빌드하면 젠킨스가  github에서 소스코드를 가져올수 있다. 

## 여러개의 리파지토리는?

위에 샘플에서는 한개의 리파지토리만 사용했다. 그런데 한번 사용한 키는 다른 리파지토리에 등록을 하려면 벌써 등록이 되잇다고 하면서 에러가 난다. 

이런 경우에는 방금 만든 deploy키를 지우고 유저에 setting에 간다. 

![]({{site_baseurl}}/assets/jenkins-github-02.png)

new ssh key를 여기에 등록을 하고 젠킨스에서는 한번 만든 인증을 여러군데에서 사용하면된다.

## Github Webhook - Jenkins 

소스코드를 push하면 사이트가 자동 빌드 되는것을 하고 싶다. 

예전에는 github service를 이용하고 젠킨스에 플러그인을 설치했는데 이 방법이 depreciate가 됬음.

새로운 webhook로 하자. 

github >> repository >> setting >> webhook >> add new webhook

![]({{site_baseurl}}/assets/github-webhook-01.png)

![]({{site_baseurl}}/assets/github-webhook-02.PNG)

혹시 안보이면 관련 플러그인을 설치하셔야 합니다.


