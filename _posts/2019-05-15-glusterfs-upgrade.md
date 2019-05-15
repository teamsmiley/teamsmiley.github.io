---
layout: post
title: 'GlusterFS Upgrade' 
author: teamsmiley
date: 2019-05-14
tags: [devops]
image: /files/covers/blog.jpg
category: {program}
---
# glusterFS Upgrade

다음 문서를 참고한다.
<https://docs.gluster.org/en/latest/Upgrade-Guide/upgrade_to_4.1/>

online 업그레이드와 offline 업그레이드 두 종류가 있으나 online upgrade를 사용할것이다.

## online upgrade
* Online upgrade is only possible with replicated and distributed replicate volumes. 우리는 distributed replicate volumes 이므로 진행해도 된다.
* 꼭 서버 한대씩 진행해야한다.
* replication을 꼭 다른서버에 하고 있는경우만  online upgrade가 가능하다.

다음 과정을 모든서버에 반복한다.

1. Stop all gluster services, either using the command below, or through other means,
```bash
killall glusterfs glusterfsd glusterd
systemctl stop glustereventsd
```
1. Stop all applications that run on this server and access the volumes via gfapi (qemu, NFS-Ganesha, Samba, etc.)
1. 새 GlusterFS를 설치한다.
```bash
yum update
yum clean all
yum install centos-release-gluster -y
yum update glusterfs-fuse
gluster --version #새버전 확인
```
1.새로 설치된 daemon을 실행한다.
```
systemctl start glusterd
systemctl enable glusterd
```
1. self-heal을 실행한다.
```bash
for i in `gluster volume list`; do gluster volume heal $i; done
```
1. no heal backlog가 있는지 확인한다.
```
gluster volume heal <volname> info
```
1. Restart any gfapi based application stopped previously in step (2)

## Post upgrade steps

1. client들도 전부 버전 업데이트를 한다.

1. enable the option
```bash
gluster volume set <volname> fips-mode-rchecksum on
```

## Upgrade procedure for clients

1. unmount 한다.
```
umount /glusterfs
```
1. Stop all applications that access the volumes via gfapi (qemu, etc.)
1. Install new Gluster
1. Mount all gluster shares
```bash
mount -t glusterfs gluster01:/gfs /glusterfs
```
1. Start any applications that were stopped previously in step 




