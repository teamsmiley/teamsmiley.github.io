---
layout: post
title: 'Computer 초기 세팅하기' 
author: teamsmiley 
date: 2016-08-12
tags: [work]
image: /files/covers/blog.jpg
category: {programe}
---

# 컴퓨터 초기 셋업

매번 컴퓨터를 셋업할때마다 시간이 오래 걸려 이번에는 적어보기로 했다.

Macosx를 써보기로 했으므로 기준은 Macosx로 하고 pc는 visual studio 사용할때만 이용함으로 가상머신 ( fusion ) 을 이용하기로 했다.
다행이 fusion라이센스가 하나 있어서 그걸 사용한다.

## 맥에서 가능한건 전부 맥에서 하자.

## 맥에 설치 한것들 
* Fusion (Vmware)
* 한글 설치
* AppStore에서 설치 
    * wunderlist 
    * marked 2
    * monosnap
    * slack 
    * kakaotalk 
    * xcode
    * kindle
    * eclise - eclpse.app 폴더를 Application폴더로 이동 
        * lombok 설치 - 다운후 더블클릭으로 실행  
        * Eclipse 메뉴에서 Window -> Preferences -> Ant -> Runtime 으로 이동한 후 Classpath 탭에서 Ant HomeEntries에 commons-net-1.4.1.jar 파일을 추가해준다.
        * bat 형식을 이클립스에서 열기 
            * preferences->general -> editors->file associoations -> add bat -> editors 
            ![]({{ site.baseurl }}/assets/eclipse_bat_filetype.png)

        * 검색에서 .svn폴더 빼기 
            * Project's Properties. >> Resource/Resource Filters >> Click Add  >>  Configure filter >>  >>
            * Filter type: Exclude all
            * Applies to: Folders
            * x All children (recursive)
            * Name, matches and type .svn
            * Click OK

            ![]({{ site.baseurl }}/assets/eclipse_search_filter.png)

* 한영 변환 ctrl+space에서  shift+space로 변경

* 직접 설치 
    * filezilla
    * dropbox 
    * skype
    * JDK 1.8 
    * eclipse 
    * chrome
    * nodejs <https://nodejs.org/en/>
        * for Frisby Test : sudo npm install -g jasmine-node
        * gulp : sudo npm install -g gulp 
        * bower : sudo npm install -g bower 
    * GitHub Desktop <https://desktop.github.com>

    * vmware 설치 
        * c드라이브를 200기가로 함.
        * cpu는 2개 메모리는 6기가정도 잇어야할듯

    * visual code 
        * markdown을 지원해서 사용 
        * 기존 서브라임 텍스트를 대체
        * extension 
            * vim 
            * auto open markdown
            * relative line number => ⌘+P : ext install vscode-relative-line-numbers




* ssh key 생성 
    * ssh-keygen -t rsa
    * 리눅스 서버들 접속 자동화를 위해 사용

* svn auto commit 
    * 커밋을 쉽게 하기 위해 스크립트를 만듬.
    * /usr/local/bin에 넣어서 어디서나 호출이 가능하게 함.
    * sudo vi /usr/local/bin/svnautocommit.sh 

```
#!/bin/bash

cd $1
svn cleanup
svn add --force .
for i in $(svn st | grep \! | awk '{print $2}'); do svn delete $i; done
svn commit . -m "brian"
```
    
* svnautocommit.sh /Path


* finder 에서 terminal을 특정 폴더에서 열기 

```
System Preferences > Keyboard > Shortcuts > Services > Files and Folders >  New Terminal at Folders
```


## vm -  윈도우 2008 R2

* 윈도우 업데이트
* visual studio
    * 플러그인 설치 
        * vsvim 
        * resharper
    * 단축키 설정 
        * package management ==> 메뉴 >> tools >> option >> Keyboard >> view.packagemanagementConsole 클릭후 alt + / , alt + . 하고 Assign
    * vim 기본 세팅 : <https://teamsmiley.github.io/2016/06/23/visual-studio-using-vi/>
    * iis express 세팅 : <https://teamsmiley.github.io/2016/08/23/iis-bad-request-error/>

* node.js  설치 
* npm install 
* npm install -g gulp
* npm install -g bower 
* git <https://git-scm.com/download/win>
* bower update
* gulp
* chrome 
* fiddler 
* IE SEC 구성 변경 <http://allsoft.co.kr/bbs/board.php?bo_table=study1_2&wr_id=6>
* 자동 로그인 -  control userpasswords2


## vm snapshot 
윈도우 업데이트가 오래 걸리므로 다 셋업해두고 스탭샷 한번 해둔다.
나중에 이상태로 복구가 가능하다.


## Time Capsule 을 구매하여 백업을 하였다. 

## 백업 방법 
* 일단 아이들의 사진을 백업하기 위하여 아내와 내가 하나의 드롭박스 아이디를 사용하여 사진을 찍으면 한곳으로 모이게 된다. 
* 드롭박스에 어느정도 양이 차면 정리하여 컴퓨터 로컬 하드 디스크에 picture폴더에 백업을 한다. 
* 타임 캡슐로 로컬 컴퓨터의 모든 파일을 백업한다.  
* 이정도면 될듯 싶다. 

 

















