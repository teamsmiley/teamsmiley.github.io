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

max_connections = 1000

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
wsrep_sst_auth                 = root:yourpass

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

mysql> update user set password = password('yourpass') where user = 'root';

mysql> flush privileges;

mysql> exit

mysql -u root -pyourpass

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
wsrep_sst_auth                 = root:yourpass

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
wsrep_sst_auth                 = root:yourpass

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

## 마무리 정리 

위에서 한것을 정리해보면 첫번째 노드에서 mysql@bootstrap을 실행했다. 이걸 하면 부트스트랩 모드가 실행이 되면서 `wsrep_cluster_address=gcomm://` 이런 모드로 할당이 되면서 나머지 노드들이 붙을수 있게 해준다.

이제 두번째 노드와 3번째 노드를 다 붙였다. 

다시 첫번째 노드에 와서 `systemctl stop mysql@bootstrap`을 해보자. 서비스가 멈추었다.

이제 정상적인 서비스를 올려보자. `systemctl start mysql`

mysql 로 접속해서 복제가 잘 되는지 확인한다. 잘 된다.

bootstrap모드는 맨 처음에 한번 하고 노드들이 다 추가가 된 후에는 

## 기존 디비 이전 
기존엔 사용하는 디비가 있으면 xtrabackup으로 백업을 받아서 새로 만든 노드에 data폴더에 넣어준후 bootstrap모드로 먼저 올려준다. 노드 2 3 번을 올려서 클러스터를 완성한후 복제가 다 되고 완료가 되면 1번째 노드를 normal모드로 올리면 기존 데이터를 다 복제한 3대의 노드가 생성이 된다.

꼭 클러스터 상태를 보고 synced일때 normal모드로 바꾼다.

```
show status like 'wsrep%';

| wsrep_local_state_comment    | Donor/Desynced

| wsrep_local_state_comment    | Synced  ==> 이렇게 되야함.

```

## 장비 고장후 대처법 

하나의 장비가 고장나면 다른 장비를 추가하여 같은 설정으로 `systemctl start mysql`을 해주면 된다.

2대의 클러스터는 추천하지 않으므로 고장시 바로 3대로 만들어야할듯 .

여전히 문제가 하나의 장비가 고장이 나면 서비스에 장애가 나긴 난다. 현재는 아이피 기반으로 프로그램이 코딩이 되있는데 그게 문제인가?

첫번째 노드 아이피가 9번이고 그 장비가 고장이 나면 새로운 노드 100번을 추가하고 디비가 다 복구가 되더라도 소스코드에서 아이피를 9번에서 100번으로 바궈줘야하는 문제가 있다. 음. 어떻게 하지? 로드 발란스를 사용해야겟다. 

proxy sql를 사용해보자.

## proxy sql 

node13

yum install proxysql2 -y

vi /etc/proxysql.cnf

admin password 변경

systemctl start proxysql 

mysql -u admin -p -h 127.0.0.1 -P 6032

SHOW DATABASES;

## add backend
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.0.9',3306);
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.0.13',3306);
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.0.14',3306);

UPDATE global_variables SET variable_value='root'
              WHERE variable_name='mysql-monitor_username';

UPDATE global_variables SET variable_value='rootdb'
              WHERE variable_name='mysql-monitor_password';

UPDATE global_variables SET variable_value='2000' WHERE variable_name IN ('mysql-monitor_connect_interval','mysql-monitor_ping_interval','mysql-monitor_read_only_interval');

SELECT * FROM global_variables WHERE variable_name LIKE 'mysql-monitor_%';

LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;

### Backend’s health check

SELECT * FROM mysql_servers;

SHOW DATABASES;

SHOW TABLES FROM monitor;

SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 6;

SELECT * FROM monitor.mysql_server_ping_log ORDER BY time_start_us DESC LIMIT 6;

LOAD MYSQL SERVERS TO RUNTIME;

Creating ProxySQL Client User

INSERT INTO mysql_users (username,password) VALUES ('root','rootdb');

LOAD MYSQL USERS TO RUNTIME;

SAVE MYSQL USERS TO DISK;

mysql -u root -prootdb -h 127.0.0.1 -P 6033


## 성능 테스트

yum install sysbench

create database sysbench;
show variables like 'socket';

### prepare

sysbench --report-interval=1 --num-threads=4 \
--max-time=20 \
/usr/share/sysbench/oltp_read_write.lua \
--table_size=10000 \
--mysql-host=localhost --mysql-port=3306 \
--db-driver=mysql --mysql-user=root --mysql-password=yourpass \
--mysql-db=sysbench --mysql-socket=/var/lib/mysql/mysql.sock --num-threads=100 \
prepare

### run
sysbench --report-interval=1 --num-threads=4 \
--max-time=20 \
/usr/share/sysbench/oltp_read_write.lua \
--table_size=10000 \
--mysql-host=localhost --mysql-port=3306 \
--db-driver=mysql --mysql-user=root --mysql-password=yourpass \
--mysql-db=sysbench --mysql-socket=/var/lib/mysql/mysql.sock --num-threads=100 \
run

### cleanup
sysbench --report-interval=1 --threads=4 --time=20 \
/usr/share/sysbench/oltp_read_write.lua \
--table_size=10000 \
--mysql-host=localhost --mysql-port=3306 \
--db-driver=mysql --mysql-user=root --mysql-password=yourpass \
--mysql-db=sysbench --mysql-socket=/var/lib/mysql/mysql.sock \
cleanup







hostgroup 0 for the master [Used for Write queries ]
hostgroup 1 for the slaves [Used for Read Queries ]



today=`date +%Y%m%d`

xtrabackup --user=root --password=rootdb --backup  --target-dir=/data/backups/${today}/ --parallel=8

