---
layout: post
title: "Percona XtraDB Cluster 8.0 without SSL"
author: teamsmiley
date: 2020-10-16
tags: [db]
image: /files/covers/blog.jpg
category: { percona }
---

# Percona XtraDB Cluster

mysql의 MMM과 같은 것이다 올리자마자 리플리케이션이랑 모든게 준비된 패키지.

8부터 ssl이 들어와서 조금 고생하고 해결하지 못햇으나 내부 망이고 하니 ssl을 빼고 진행하는것으로 했다.

node 3개를 준비했다.

node 1 : 192.168.0.101
node 2 : 192.168.0.102
node 3 : 192.168.0.103

전체노드 selinux를 꺼두고 방화벽도 다꺼두고 작업하자.

centos 7을 기본 설치했다.

이후부터 진행해보자.

참고 <https://www.percona.com/doc/percona-repo-config/percona-release.html#example-all-steps-for-installing-a-specific-percona-product>

## node1

```bash
yum update -y

yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm -y

yum update -y

percona-release enable-only pxc-80 release
percona-release enable tools release
yum install percona-xtradb-cluster -y
```

```bash
systemctl start mysql
grep 'temporary password' /var/log/messages
> JYUOihedh9?m

mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Password'; # 비번 변경

service mysql stop
```

이러면 기본 디비 데이터들이 다 만들어진다.

ini를 만들자.

```bash
cat >/etc/my.cnf<<EOF
[client]
socket=/var/lib/mysql/mysql.sock

[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Binary log expiration period is 604800 seconds, which equals 7 days
binlog_expire_logs_seconds=604800

######## wsrep ###############
# Path to Galera library
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://192.168.0.101,192.168.0.102,192.168.0.103

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# Slave thread to use
wsrep_slave_threads=8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
wsrep_node_address=192.168.0.101
# Cluster name
wsrep_cluster_name=pxc-cluster8

#If wsrep_node_name is not specified,  then system hostname will be used
#wsrep_node_name=pxc-cluster-node-1

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

pxc-encrypt-cluster-traffic=OFF
EOF
```

```bash
systemctl start mysql@bootstrap.service
mysql -u root -p
show status like 'wsrep%';
```

```
| wsrep_cluster_conf_id            | 1                                                                                                                                              |
| wsrep_cluster_size               | 1
```

## node2

이제 노드 2번을 설치하자. 1번과 다르게 ini만 작성하고 mysql을 시작하면 자동으로 모든 내용을 다운받는다.

```bash
percona-release enable-only pxc-80 release
percona-release enable tools release
yum install percona-xtradb-cluster -y
```

```bash
cat >/etc/my.cnf<<EOF
[client]
socket=/var/lib/mysql/mysql.sock

[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Binary log expiration period is 604800 seconds, which equals 7 days
binlog_expire_logs_seconds=604800

######## wsrep ###############
# Path to Galera library
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://192.168.0.101,192.168.0.102,192.168.0.103

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# Slave thread to use
wsrep_slave_threads=8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
wsrep_node_address=192.168.0.102
# Cluster name
wsrep_cluster_name=pxc-cluster8

#If wsrep_node_name is not specified,  then system hostname will be used
#wsrep_node_name=pxc-cluster-node-1

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

pxc-encrypt-cluster-traffic=OFF
EOF
```

```bash
systemctl enable mysql
systemctl start mysql

mysql -u root -p

show status like 'wsrep%';
```

ini만 설정하고 나면 자동으로 전부 다운받아서 싱크한다.

## node3

```bash
percona-release enable-only pxc-80 release
percona-release enable tools release
yum install percona-xtradb-cluster -y
```

```bash
cat >/etc/my.cnf<<EOF
[client]
socket=/var/lib/mysql/mysql.sock

[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# Binary log expiration period is 604800 seconds, which equals 7 days
binlog_expire_logs_seconds=604800

######## wsrep ###############
# Path to Galera library
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://192.168.0.101,192.168.0.102,192.168.0.103

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# Slave thread to use
wsrep_slave_threads=8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
wsrep_node_address=192.168.0.103
# Cluster name
wsrep_cluster_name=pxc-cluster8

#If wsrep_node_name is not specified,  then system hostname will be used
#wsrep_node_name=pxc-cluster-node-1

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

pxc-encrypt-cluster-traffic=OFF
EOF
```

```bash
systemctl enable mysql
systemctl start mysql
mysql -u root -p

show status like 'wsrep%';
# 결과
| wsrep_local_state_comment        | Synced
| wsrep_cluster_conf_id            | 3                                                                                                                                              |
| wsrep_cluster_size               | 3
```

ini만 설정하고 나면 자동으로 전부 다운받아서 싱크한다.

## 확인하자.

node1에서 db를 만들어보자.

node1에서 create database test;

show databases;

이제 node2,node3 에서 해보자.

show databases;

test 디비가 모든곳에 있으면 된다.

이제 node3에서 디비를 생성해보고 다른 곳에도 생기는지 확인하자.

잘 생긴다.

## node1 클러스터에 포함 하기

첫번째 노드에서 mysql@bootstrap을 실행했다. 이걸 하면 부트스트랩 모드가 실행이 되면서 `wsrep_cluster_address=gcomm://` 이런 모드로 할당이 되면서 나머지 노드들이 붙을수 있게 해준다.

이제 두번째 노드와 3번째 노드를 다 붙였다.

이제 1번 노드를 정상적인 클러스터에 포함을 시키자.

```bash
systemctl stop mysql@bootstrap
systemctl start mysql
```

mysql 로 접속해서 복제가 잘 되는지 확인한다. 잘 된다.

## 장비 고장후 대처법

하나의 장비가 고장나면 다른 장비를 추가하여 같은 설정으로 `systemctl start mysql`을 해주면 된다.

2대의 클러스터는 추천하지 않으므로 고장시 바로 3대로 만들어야할듯 .

## todo

여전히 문제가 하나의 장비가 고장이 나면 서비스에 장애가 나긴 난다.

sql proxy로 해결이 가능하다는데.. 나중에 해보자.
