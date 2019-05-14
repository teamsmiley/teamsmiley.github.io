---
layout: post
title: 'GlusterFS' 
author: teamsmiley
date: 2019-05-14
tags: [devops]
image: /files/covers/blog.jpg
category: {program}
---
# glusterFS

https://docs.gluster.org/en/latest/Administrator%20Guide/Setting%20Up%20Clients/

GlusterFS라고 여러개의 서버를 이어서 하나의 서버(스토리지)처럼 보이는게 하는 것이다. 

## server setup
```
192.168.0.196 gluster00 => Arbiter 
192.168.0.197 gluster01
192.168.0.198 gluster02 
```

전체 서버에 방화벽을 전부 끄고 시작하자.

전체 서버에 1기가 하드를 추가하고 Linux Volume Manager 을 사용하여 xfs로 설정 

## GlusterFS 
* Trusterd storage pool
* Node
* Brick
* volume

## Volume Type
* Distributed
* Replicated
* Distributed-Replicated

## install
vi /etc/hosts
```
192.168.0.196 node196 gluster00
192.168.0.197 node197 gluster01
192.168.0.198 node198 gluster02
```

```bash
yum install epel-release -y
yum install centos-release-gluster -y
yum install wget -y
yum install glusterfs-server -y
systemctl start glusterd
systemctl enable glusterd
```

## storage pool 등록

gluster00에서 다음 명령어 
```bash
gluster peer probe gluster01
gluster peer probe gluster02
```

gluster00에서 다음처럼 확인
```bash
gluster peer status

> Number of Peers: 1
> Hostname: gluster01
> Uuid: 5d065872-b794-451a-bbbd-66d698272c7e
> State: Peer in Cluster (Connected)
```

## Distribute-Replicate Volume Setup

![](https://cloud.githubusercontent.com/assets/10970993/7412402/23a17eae-ef60-11e4-8813-a40a2384c5c2.png)

00번 서버에서
```
mkdir -p /data/glusterfs/vol01/arbiter
```

01 02번 서버에서 
```
mkdir -p /data/glusterfs/vol01/brick
```

이제 볼륨을 만들어보자. 

```bash
gluster volume create gfs replica 2 arbiter 1 gluster01:/data/glusterfs/vol01/brick gluster02:/data/glusterfs/vol01/brick gluster00:/data/glusterfs/vol01/arbiter01
#gluster volume create gfs replica 2 arbiter 1 gluster{01..02}:/data/glusterfs/vol01/brick gluster00:/data/glusterfs/vol01/arbiter

gluster volume start gfs
```

## 다른서버에서 사용하자.
중요한것은 arbiter노드는 매핑에 사용하지 않는다. 
vi /etc/hosts
```
192.168.0.196 node196 gluster00
192.168.0.197 node197 gluster01
192.168.0.198 node198 gluster02
```

```bash
mkdir /glusterfs
yum install glusterfs-fuse -y
mount -t glusterfs gluster01:/gfs /glusterfs
#mount -t glusterfs gluster02:/gfs /glusterfs

ls /glusterfs
touch /glusterfs/file{00..99}
ls /glusterfs

vi /etc/fstab
> gluster01:/gfs /glusterfs glusterfs defaults,_netdev 0 0
```

## storage 추가 
```bash
gluster volume add-brick gfs gluster03:/data/glusterfs/vol01/brick gluster04:/data/glusterfs/vol01/brick gluster00:/data/glusterfs/vol01/arbiter02
```

## 추가 커맨드 
```bash
gluster volume info 
gluster volume start 
gluster volume stop
gluster volume delete
```

https://wiki.centos.org/HowTos/GlusterFSonCentOS





