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

### mssql 프로세스 확인
```
docker ps 
```
프로세스가 보이면 된다

## 뷰어 

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
# 플러그인 설치
CTRL+SHIFT+P
```

install extension

mssql을 선택한다.

### vscode를 열어서 파일을 하나 만들고 파일명을 aaa.sql로 저장한다.

sql을 쓰면 인텔리센스가 나오는지 확인한다.

### 디비에 접속해보자.

```
CTRL+SHIFT+P
sqlcon enter
create connect profile
```
enter
servername enter
id 입력후 엔터 sa를 사용
password 입력 엔터 
프로파일명 입력 엔터 

생성이 되면 command + , 를 누르면 유저 세팅에서 내용이 보인다. 

```
"mssql.connections": [
    {
      "server": "localhost",
      "database": "",
      "authenticationType": "SqlLogin",
      "user": "sa",
      "password": "Password12#$",
      "emptyPasswordInput": false,
      "savePassword": true,
      "profileName": "local"
    }
  ]
```

아래쪽 하단을 보면 커넥션이 됬다고 표시가 됩니다.

### 추가 설정하기 
<https://github.com/Microsoft/vscode-mssql#options>

이 링크를 확인하시면 사용 가능한 옵션을 확인할수 있다. 

### 샘플 디비 만들기 

vs code에서 aaa.sql을 열고 다음처럼 입력합니다.

sqlCreateDatabase 엔터 

코드 스닙핏이 코드를 완성해줍니다. 

계속해서 간단히 디비명만 변경해줍니다.

```sql
-- Create a new database called 'Sample'
-- Connect to the 'master' database to run this snippet
USE master
GO
-- Create the new database if it does not exist already
IF NOT EXISTS (
  SELECT name
FROM sys.databases
WHERE name = N'Sample'
)
CREATE DATABASE Sample
GO
```

```
command + shift + p 
mssql: execute sql 
```
또는 
```
command + shift + p 
```


### 디비가 만들어져 있는지 확인해봅니다.

기존 쿼리를 지우고 다음처럼 입력한다.
```
select * from sys.databases 
command + shift + e
```

결과창에 sample이 있으면 성공 

### 추가 팁

* 쿼리를 선택을 한 상태에서 command + shift + e 를 하면 선택된 부분만 실행된다.

* 대소문자를 구분을 한다.

