--- 
layout: post 
title: "tensorflow - 1" 
date: 2017-02-15 00:00  
author: teamsmiley 
tags: [deep learning]
image: /files/covers/blog.jpg
category: {AI}
---

# tensorflow 윈도우에서 사용  

## 윈도우에서 docker로 진행하려고 한다. 

* 윈도우 7 8 에서는 docker tools 로 진행 
* 윈도우 10에서는 docker-for-windows로 진행하는게 좋다고 한다. 
    * 도커포윈도우 (https://docs.docker.com/docker-for-windows)
    * 도커포윈도우는 최신 버전의 Hyper-V 프로그램을 사용 현재 없으면 제어판에서 설치해줘야함. 

![]({{ site.baseurl }}/assets/hyper-v.png)


## docker-for-windows 설치 

## Hyper-v 설치 

## cmd 열어서 docker info  확인한다. 

## powershell을 어드민 권한으로 연다

* Set-ExecutionPolicy Unrestricted -> All
* Install-PackageProvider ContainerImage -Force 

## 텐서 플로어 도커 실행 처음 실행시 없으면 다운받는다. 
```
docker run -it -p 8888:8888 gcr.io/tensorflow/tensorflow
```

> 잘안되면 이거 하기전에 https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/live/windows-server-container-tools/Update-ContainerHost/Update-ContainerHost.ps1 다운받아 저장후 powershell에서 실행한번 해준다..중간에 에러떨어지면 무시 한후 위에작업을 한번 더 해보자.

다 설치후 powershell 결과를 보면 아래와 비슷한 내용이 있다. 

http://localhost:8888/?token=788169978349801953fabc797f746384911b3d0eebad1934

웹브라우져에 붙여 넣으면 된다. 

이제 도커로 들어가보자. 
ctrl c로 멈추고 

다음 명령어 실행 

 docker run -it gcr.io/tensorflow/tensorflow bash

bash가 보일것이다.

이 도커에에는 python 2 와 3이 동시에 설치되잇다. 
기본으로 python2로 연동되 있다. 
```
python -c 'import os; import inspect; import tensorflow; print(os.path.dirname(inspect.getfile(tensorflow)))'
```

이제 다시 웹브라우저로 가서 파이선 코딩을 해보자. 

docker run -it -p 8888:8888 gcr.io/tensorflow/tensorflow

url 을 복사해다 웹브라우저에 붙여넣기 하고 
새로 파일 만들기를 한후 
다음 코드 적용

print ("hello");
실행 버튼을 누르면 hello가 나온다.

이상 여기서 끝














