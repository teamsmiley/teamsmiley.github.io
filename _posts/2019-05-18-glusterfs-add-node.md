---
layout: post
title: 'GlusterFS Add Node' 
author: teamsmiley
date: 2019-05-18
tags: [devops]
image: /files/covers/blog.jpg
category: {program}
---
# GlusterFS Add Node 

## Edit Hosts
* from gluster00 gluster05 gluster06

vi /etc/hosts

```bash
192.168.0.201 node201 gluster05
192.168.0.202 node202 gluster06
```

## add harddisk on gluster05 and 06 
```
fdisk -l
lsblk
fdisk /dev/sdb
> d / enter
pvcreate /dev/sdb
vgcreate vg_1  /dev/sdb

lvcreate -l +100%FREE -n data vg_1
mkfs -t xfs /dev/vg_1/data

blkid /dev/vg_1/data
> /dev/vg_1/data: UUID="76b6f3e1-a58c-4d62-924e-20f952d63fa7" TYPE="xfs"

mkdir /data

vi /etc/fstab
> UUID=76b6f3e1-a58c-4d62-924e-20f952d63fa7 /data xfs defaults 0 0 

mount -a
```

## Install Gluster
```bash
yum install epel-release -y
yum install centos-release-gluster -y
yum install wget -y
yum install glusterfs-server -y
systemctl start glusterd
systemctl enable glusterd
```

## Add storage pool (on gluster00)
```
gluster peer probe gluster05
gluster peer probe gluster06
```

## storage pool 확인
gluster00에서 다음처럼 확인
```
gluster peer status
```

## create arbiter folder on gluster00
```
mkdir -p /data/glusterfs/vol01/arbiter03
```

## create brick on gluster05 gluster06
```
mkdir -p /data/glusterfs/vol01/brick
```

## create volume
```
gluster volume add-brick gfs gluster05:/data/glusterfs/vol01/brick gluster06:/data/glusterfs/vol01/brick gluster00:/data/glusterfs/vol01/arbiter03
```

## 확인
```
mkdir /glusterfs
yum install glusterfs-fuse -y
mount -t glusterfs gluster01:/gfs /glusterfs
df -h
> Filesystem             Size  Used Avail Use% Mounted on
> /dev/sda2              204G  4.1G  190G   3% /
> devtmpfs                16G     0   16G   0% /dev
> tmpfs                   16G     0   16G   0% /dev/shm
> tmpfs                   16G   18M   16G   1% /run
> tmpfs                   16G     0   16G   0% /sys/fs/cgroup
> /dev/sda1              9.8G  174M  9.1G   2% /boot
> tmpfs                  3.2G     0  3.2G   0% /run/user/0
> /dev/mapper/vg_1-data  932G  4.5G  927G   1% /data
> gluster01:/gfs         2.8T  1.3T  1.5T  48% /glusterfs
```

1Tera가 늘어난것을 알수 있다.



