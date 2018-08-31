---
layout: post
title: 'mssql 2017 docker on macosx' 
author: teamsmiley
date: 2018-08-30
tags: [mssql,docker,docker-machine]
image: /files/covers/blog.jpg
category: {macosx,docker}
---

# mssql 2017 docker

## install docker

https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-2017

## install mssql 

### install mssql 2016 docker image
```
sudo docker pull microsoft/mssql-server-linux:2017-latest
```

### mssql run

```
sudo docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Password12#$' \
   -p 1433:1433 --name mssql \
   -d microsoft/mssql-server-linux:2017-latest
```

* Specify your own strong password that is at least 8 characters and meets the SQL Server password requirements. Required setting for the SQL Server image.


## 뷰어 설치 - vscode

https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-develop-use-vscode?view=sql-server-2017

### install vscode & openssl

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install openssl
brew upgrade openssl
ln -s /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib /usr/local/lib/
ln -s /usr/local/opt/openssl/lib/libssl.1.0.0.dylib /usr/local/lib/
code .
CTRL+SHIFT+P
mssql
```

## vscode를 열어서 파일을 하나 만들고 파일명을 aaa.sql로 저장한다.

sql을 쓰면 인텔리센스가 나오는지 확인한다. 



