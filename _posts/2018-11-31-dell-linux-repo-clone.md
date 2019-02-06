---
layout: post
title: 'internal centos yum repo, dell repo , docker repo' 
author: teamsmiley
date: 2018-11-30
tags: [devops]
image: /files/covers/blog.jpg
category: {linux}
---

# 내부네트워크에서 사용 하기  위해 repo를 mirror 한다. 

## 서버 준비 

도커로 nginx를 사용함 

### get-docker.sh
```
mkdir -p /data/mirror/docker
cd /data/mirror/docker
curl -fsSL https://get.docker.com -o get-docker.sh
```

### yum repo mirror 
```
mkdir -p /data/mirror/docker
cd /data/mirror/docker
wget --mirror --convert-links --no-parent https://download.docker.com/linux/centos/7/x86_64/
mv download.docker.com/linux .
wget --no-parent https://download.docker.com/linux/centos/docker-ce.repo
wget --no-parent https://download.docker.com/linux/centos/gpg
```

### modify docker-ce.repo file
```
cd /data/mirror/docker/linux/centos/7/
vi docker-ce.repo
```

### docker-ce.repo file contents
```
cat docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=http://yum/docker/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=http://yum/docker/linux/centos/7/gpg
```

### docker-compose
```
mkdir -p /data/mirror/docker
wget -p https://github.com/docker/compose/releases/download/1.22.0/docker-compose-Linux-x86_64 -O /data/mirror/docker/docker-compose
```

### change permission
```
mkdir -p /data/mirror/docker
chown -R root:root /data/mirror/docker
chmod -R 755 /data/mirror/docker
```

vi /data/git/docker/docker-compose.yml
```yml
---
version: '3.3'

services:
  nginx:
    container_name: nginx
    image: nginx:1.14
    hostname: nginx
    environment:
      TX: "America/Los_Angeles"
    restart: always
    volumes:
      - "/data/mirror:/usr/share/nginx/html"

    ports:
      - "80:80"
```

## docker up
```
cd /data/git/docker/
docker-compose up -d 
```


## yum repo mirror
```bash
mkdir -p /data/mirror/centos_vault
mkdir -p /data/mirror/centos
mkdir -p /data/mirror/epel

# 5
rsync -avSHP --progress --exclude "local*" --exclude "isos" -e ssh root@yum.publishapi.com:/data/mirror/centos_vault/ /data/mirror/centos_vault >> rsync-repo.log

#7
rsync -avSHP --progress --exclude "local*" --exclude "isos" -e ssh root@yum.publishapi.com:/data/mirror/centos/ /data/mirror/centos >> rsync-repo.log

# epel 
rsync -avSHP --progress --exclude "local*" --exclude "isos" -e ssh root@yum.publishapi.com:/data/mirror/epel/ /data/mirror/epel >> rsync-repo.log
```

## dell linux repo mirror
```bash
mkdir -p /data/mirror/dell
cd /data/mirror/dell
rsync -avHz linux.dell.com::repo/hardware .
chown -R root:root /data/mirror/dell
chmod -R 755 /data/mirror/dell
```
## modify bootstrap.sh to use c1br-yum repo
```
cd /data/mirror/dell/hardware/DSU_16.01.00
cp bootstrap.sh bootstrap.sh.orig
cat bootstrap.sh.orig | sed -E 's/SERVER=/\$SERVER=/g' | sed -E 's/REPO_URL=/\$REPO_URL=/gi' | sed -E 's/os_dependent\//os_dependent\/\$VERSION\//g' > bootstrap.sh
```

## docker repo mirror

### get-docker.sh
```
mkdir -p /data/mirror/docker
cd /data/mirror/docker
curl -fsSL https://get.docker.com -o get-docker.sh
```

### yum repo mirror 
```
mkdir -p /data/mirror/docker
cd /data/mirror/docker
wget --mirror --convert-links --no-parent https://download.docker.com/linux/centos/7/x86_64/
mv download.docker.com/linux .
wget --no-parent https://download.docker.com/linux/centos/docker-ce.repo
wget --no-parent https://download.docker.com/linux/centos/gpg
```

### modify docker-ce.repo file
```
cd /data/mirror/docker/linux/centos/7/
vi docker-ce.repo
```

### docker-ce.repo file contents
```
cat docker-ce.repo 
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=http://yum/docker/linux/centos/7/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=http://yum/docker/linux/centos/7/gpg
```

### docker-compose
```
mkdir -p /data/mirror/docker
wget -p https://github.com/docker/compose/releases/download/1.22.0/docker-compose-Linux-x86_64 -O /data/mirror/docker/docker-compose
```

### change permission
```
chown -R root:root /data/mirror/docker
chmod -R 755 /data/mirror/docker
```



