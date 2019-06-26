---
layout: post
title: '게시판 만들기-01' 
author: teamsmiley
date: 2019-04-15
tags: [devops]
image: /files/covers/blog.jpg
category: {blog}
---

# angular 7 + web api (dotnet core 2.2) 개발환경구축 

## 패키지 매니저 설치 

* windows - chocolatey (https://chocolatey.org/install)
```bash
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

* macos - brew (https://brew.sh)
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update && brew upgrade
brew tap caskroom/cask
```

## window user

* visual studio 2019 or 2017 (https://visualstudio.microsoft.com/)

* vscode 
```bash
choco install vscode -y
```

* dotnet core 2.2 sdk 
```bash
choco install dotnetcore-sdk -y
```

* git bash 
```bash
choco install git -y
```

* postman
```bash
brew cask install postman
```

* nodejs (https://nodejs.org/ko/download/)
```bash
choco install nodejs-lts -y
```

* 추가 npm package
```bash
npm install -g @angular/cli
```

* vs code plugin install 
  * Angular Language Service

## macos user

* dotnet core 2.2 sdk 
```bash
brew cask install dotnet
```

* git 
```bash
git --version # 기존 버전 확인
brew install git
brew link --force git
git --version # 새버전 확인
```

* vscode 
```bash
brew cask install visual-studio-code
```

* postman
```bash
brew cask install postman
```

* nodejs
```bash
brew install node
```

* 추가 npm package
```bash
npm install -g @angular/cli
```

## vs code plugin
Angular Language Service 를 설치한다. 







