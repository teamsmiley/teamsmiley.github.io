---
layout: post
title: 'percona-xtradb-cluster' 
author: teamsmiley
date: 2020-08-17
tags: [xtradb]
image: /files/covers/blog.jpg
category: {db}
---

# Percona XtraDB Cluster

node 3개를 준비했다.

node 9 : 192.168.0.9
node 13 : 192.168.0.13
node 14 : 192.168.0.14

기준이 되는 노드를 먼저 설치를 하자.

전체노드 selinux를 꺼두고 방화벽도 다꺼두고 작업하자. 

centos 7을 기본 설치했다.

이후부터 진행해보자.

## node 9

```bash
yum update -y

yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm -y

yum install Percona-XtraDB-Cluster-56 -y
```

ini를 만들자.
```
cat >>/etc/my.cnf<<EOF
[mysqld]

max_connections = 10000

log_bin

binlog_format                  = ROW
innodb_buffer_pool_size        = 100M
innodb_flush_log_at_trx_commit = 0
innodb_flush_method            = O_DIRECT
innodb_log_files_in_group      = 2
innodb_log_file_size           = 20M
innodb_file_per_table          = 1
datadir                        = /var/lib/mysql

wsrep_cluster_address          = gcomm://192.168.0.9,192.168.0.13,192.168.0.14
wsrep_provider                 = /usr/lib64/galera3/libgalera_smm.so

wsrep_slave_threads            = 8
wsrep_cluster_name             = Cluster

wsrep_node_name                = Node09
wsrep_node_address             = 192.168.0.9

wsrep_sst_method               = xtrabackup-v2
wsrep_sst_auth                 = root:rootdb

default_storage_engine         = InnoDB

#pxc_strict_mode                = ENFORCING

innodb_autoinc_lock_mode       = 2

log-error                      = /var/log/mysql/error.log

long_query_time                = 5
slow_query_log                 = 1
slow_query_log_file            = /var/log/mysql/mysql_slow.log

#log                           = /var/log/mysql/query.log

[mysqld_safe]
pid-file = /run/mysqld/mysql.pid
#syslog

!includedir /etc/my.cnf.d

EOF
```

로그를 한번 켜두고 서비스를 시작해보자.

```
tail -f /var/log/message
```

5.6버전의 경우 비번은 없다. 그러므로 다음처럼 로그인한다.

```
systemctl start mysql@bootstrap

mysql 
```

이제 로그인후 비번을 설정하자.
```bash
mysql> use mysql;

mysql> update user set password = password('rootdb') where user = 'root';

mysql> flush privileges;

mysql> exit

mysql -u root -prootdb

show status like 'wsrep%';
```

## node13

이제 노드 2번을 설치하자. 1번과 다르게 ini만 작성하고 mysql을 시작하면 자동으로 모든 내용을 다운받는다.

```
yum install Percona-XtraDB-Cluster-56
```

```
cat >>/etc/my.cnf<<EOF
[mysqld]

max_connections = 10000

log_bin

binlog_format                  = ROW
innodb_buffer_pool_size        = 100M
innodb_flush_log_at_trx_commit = 0
innodb_flush_method            = O_DIRECT
innodb_log_files_in_group      = 2
innodb_log_file_size           = 20M
innodb_file_per_table          = 1
datadir                        = /var/lib/mysql

wsrep_cluster_address          = gcomm://192.168.0.9,192.168.0.13,192.168.0.14
wsrep_provider                 = /usr/lib64/galera3/libgalera_smm.so

wsrep_slave_threads            = 8
wsrep_cluster_name             = Cluster

wsrep_node_name                = Node13
wsrep_node_address             = 192.168.0.13

wsrep_sst_method               = xtrabackup-v2
wsrep_sst_auth                 = root:rootdb

default_storage_engine         = InnoDB

#pxc_strict_mode                = ENFORCING

innodb_autoinc_lock_mode       = 2

log-error                      = /var/log/mysql/error.log

long_query_time                = 5
slow_query_log                 = 1
slow_query_log_file            = /var/log/mysql/mysql_slow.log

#log                           = /var/log/mysql/query.log

[mysqld_safe]
pid-file = /run/mysqld/mysql.pid
#syslog

!includedir /etc/my.cnf.d

EOF
```

```
systemctl enable mysql

systemctl start mysql 
```
ini만 설정하고 나면 자동으로 전부 다운받아서 싱크한다.

## node14


`yum install Percona-XtraDB-Cluster-56`

```
cat >>/etc/my.cnf<<EOF
[mysqld]

max_connections = 10000

log_bin

binlog_format                  = ROW
innodb_buffer_pool_size        = 100M
innodb_flush_log_at_trx_commit = 0
innodb_flush_method            = O_DIRECT
innodb_log_files_in_group      = 2
innodb_log_file_size           = 20M
innodb_file_per_table          = 1
datadir                        = /var/lib/mysql

wsrep_cluster_address          = gcomm://192.168.0.9,192.168.0.13,192.168.0.14
wsrep_provider                 = /usr/lib64/galera3/libgalera_smm.so

wsrep_slave_threads            = 8
wsrep_cluster_name             = Cluster

wsrep_node_name                = Node14
wsrep_node_address             = 192.168.0.14

wsrep_sst_method               = xtrabackup-v2
wsrep_sst_auth                 = root:rootdb

default_storage_engine         = InnoDB

#pxc_strict_mode                = ENFORCING

innodb_autoinc_lock_mode       = 2

log-error                      = /var/log/mysql/error.log

long_query_time                = 5
slow_query_log                 = 1
slow_query_log_file            = /var/log/mysql/mysql_slow.log

#log                           = /var/log/mysql/query.log

[mysqld_safe]
pid-file = /run/mysqld/mysql.pid
#syslog

!includedir /etc/my.cnf.d

EOF
```

```
systemctl enable mysql
systemctl start mysql 
```

ini만 설정하고 나면 자동으로 전부 다운받아서 싱크한다.

## 확인하자.

node 9에서 db를 만들어보자.

node 9에서 create database test;

show databases;

이제 노드 13에서 해보자.

show databases;

노드 14에서도 

show databases;

test 디비가 모든곳에 있으면 된다.

이제 node14에서 디비를 생성해보고 다른 곳에도 생기는지 확인하자.

잘 생긴다.

## todo 

haproxy를 사용해서 로드발란스를 해야한다.

proxy sql를 사용해보자.






