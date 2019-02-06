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

### install mssql 2017 docker image
```
sudo docker pull microsoft/mssql-server-linux:2017-latest
```

### mssql run
데이터 저장할 폴더를 만들고 사용합니다. 저는 Users/<your-id>/Desktop/mssql/data 여기로 정했습니다.

```
mkdir -p ~/Desktop/mssql

docker run --name mssql -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=StrongPassw0rd' -p 1433:1433  -d microsoft/mssql-server-linux:latest

docker logs mssql
```

### mssql 프로세스 확인
```bash
docker ps # 프로세스가 보이면 된다
docker stop mssql # 멈출때
docker start mssql # 시작할때
docker rm mssql # 지울때 
```

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
      "password": "",
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

vs code에서 sample.sql을 열고 다음처럼 입력합니다.

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

## 디비 임포트 
https://github.com/Microsoft/sql-server-samples/releases 에서 AdventureWorks2017.bak 를 다운받는다.

```bash
cd ~/Desktop/mssql
brew install wget
# bak file download
wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2017.bak

# create folder into container
docker exec -it mssql mkdir /var/opt/mssql/backup 

# copy into container
docker cp ~/Desktop/mssql/AdventureWorks2017.bak mssql:/var/opt/mssql/backup/AdventureWorks2017.bak

# vscode에서 복구 명령어를 실행하자.
code import.sql
```

로지컬 이름을 알야내야하므로 다음 쿼리를 선택후 ctrl + shift + e 
```sql
RESTORE filelistonly FROM DISK = '/var/opt/mssql/backup/AdventureWorks2017.bak'
```
AdventureWorks2017

AdventureWorks2017_log

두개를 알아냈음  다음 쿼리를 실행해서 복구한다. 

```sql
RESTORE DATABASE AdventureWorks2017 FROM DISK = '/var/opt/mssql/backup/AdventureWorks2017.bak' WITH MOVE 'AdventureWorks2017' TO '/var/opt/mssql/backup/AdventureWorks2017.mdf', MOVE 'AdventureWorks2017_Log' TO '/var/opt/mssql/backup/AdventureWorks2017.ldf', REPLACE
```

복구 완료 

sample.sql에서 디비리스트를 뽑아보면 adventureworks가 있으면 성공 

