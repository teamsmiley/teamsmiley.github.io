---
layout: post
title: 'kubernetes mysql replication'
author: teamsmiley
date: 2020-01-16
tags: [kubernetes]
image: /files/covers/blog.jpg
category: { kuberentes }
---

mysql을 쿠버네티스에 배포 연습

다음 방식이 있다.

1. statefulset 을 이용한 자동 배포
1. 수동으로 master slave를 설정하는 배포

## statefulset을 사용해서 배포를 하면 다음처럼 하면된다.

Run a Replicated Stateful Application

<https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/>

pv가 dynamic provision이 되면 아주 잘 된다.

pv는 ceph를 설치해서 해결햇음. 잘 된다.

스토리지가 좋으면 되는데 네트워크로 묶어서 사용하는 ceph다 보니 해보니 퍼포먼스가 로컬 하드디스크를 쓰는것보다 잘 안나오는듯 보인다.

## 수동 배포

kubernetes 노드가 4대가 있고 mysql을 각각 노드에 띄우고 모든 pod는 자신의 하드디스크를 스토리지로 사용한다.

일단 이것이 나에게는 퍼포먼스 문제를 해결할수 있어서 조금더 나은 선택으로 보여서 작업햇다.

### server 구성 테이블

|          |         |              |               |
| -------- | ------- | ------------ | ------------- |
| NodeName | Type    | Node IP      | serviceIP     |
| node 11  | master  | 192.168.0.11 | 192.168.0.111 |
| node 12  | slave01 | 192.168.0.12 | 192.168.0.112 |
| node 13  | slave02 | 192.168.0.13 | 192.168.0.112 |
| node 14  | slave03 | 192.168.0.14 | 192.168.0.112 |

### 설정파일 만들기

configmap.yml

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: master.cnf
  namespace: staging
data:
  master.cnf: |
    # Apply this config only on the master.
    [mysqld]
    log-bin=mysql-bin.log
    server-id = 1
    sync_binlog=1
    #default_authentication_plugin= mysql_native_password  # mysql 8만 해당 다른버전은 삭제할것

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: slave01.cnf
  namespace: staging
data:
  slave01.cnf: |
    # Apply this config only on slaves.
    [mysqld]
    server-id = 2
    slave-skip-errors=all
    #default_authentication_plugin= mysql_native_password  # mysql 8만 해당 다른버전은 삭제할것

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: slave02.cnf
  namespace: staging
data:
  slave02.cnf: |
    # Apply this config only on slaves.
    [mysqld]
    server-id = 3
    slave-skip-errors=all
    #default_authentication_plugin= mysql_native_password  # mysql 8만 해당 다른버전은 삭제할것

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: slave03.cnf
  namespace: staging
data:
  slave03.cnf: |
    # Apply this config only on slaves.
    [mysqld]
    server-id = 4
    slave-skip-errors=all
    #default_authentication_plugin= mysql_native_password  # mysql 8만 해당 다른버전은 삭제할것
```

master.yml

```yml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.beta.kubernetes.io/gid: '0'
  labels:
    type: local
  name: staging-master-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/staging/mysql-master
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  volumeMode: Filesystem

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    type: local
  name: staging-master-pvc
  namespace: staging
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local
  storageClassName: manual
  volumeMode: Filesystem
  volumeName: staging-master-pv
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10G

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-master
  namespace: staging
  labels:
    app: auth
spec:
  selector:
    matchLabels:
      app: mysql-master
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-master
    spec:
      nodeName: node11 #여기서 노드 할당
      containers:
        - image: mysql:8
          imagePullPolicy: Always
          name: mysql-master
          args:
            - '--default-authentication-plugin=mysql_native_password' # mysql 8만 해당 다른버전은 삭제할것
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: YOUR_PASSWORD
          ports:
            - containerPort: 3306
              name: mysql-master
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
            - name: config-volume
              mountPath: /etc/mysql/conf.d
      volumes:
        - name: mysql-volume
          persistentVolumeClaim:
            claimName: staging-master-pvc
        - name: config-volume
          configMap:
            name: master.cnf

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-master
  namespace: staging
  labels:
    app: mysql-master
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.111
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql-master
```

slave01.yml

```yml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.beta.kubernetes.io/gid: '0'
  labels:
    type: local
  name: staging-slave01-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/staging/mysql-slave01
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  volumeMode: Filesystem

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    type: local
  name: staging-slave01-pvc
  namespace: staging
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local
  storageClassName: manual
  volumeMode: Filesystem
  volumeName: staging-slave01-pv
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10G

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-slave01
  namespace: staging
  labels:
    app: mysql-slave
spec:
  selector:
    matchLabels:
      app: mysql-slave
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-slave
    spec:
      nodeName: node12 #여기서 노드 할당
      containers:
        - image: mysql:8
          imagePullPolicy: Always
          name: mysql-slave01
          args:
            - '--default-authentication-plugin=mysql_native_password' # mysql 8만 해당 다른버전은 삭제할것
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: YOUR_PASSWORD
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
            - name: config-volume
              mountPath: /etc/mysql/conf.d
      volumes:
        - name: mysql-volume
          persistentVolumeClaim:
            claimName: staging-slave01-pvc
        - name: config-volume
          configMap:
            name: slave01.cnf

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-slave
  namespace: staging
  labels:
    app: mysql-slave
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.87
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: mysql-slave
```

slave02.yml

```yml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.beta.kubernetes.io/gid: '0'
  labels:
    type: local
  name: staging-slave02-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/staging/mysql-slave02
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  volumeMode: Filesystem

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    type: local
  name: staging-slave02-pvc
  namespace: staging
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local
  storageClassName: manual
  volumeMode: Filesystem
  volumeName: staging-slave02-pv
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10G

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-slave02
  namespace: staging
  labels:
    app: auth
spec:
  selector:
    matchLabels:
      app: mysql-slave
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-slave
    spec:
      nodeName: node13 #여기서 노드 할당
      containers:
        - image: mysql:8
          imagePullPolicy: Always
          name: mysql-slave02
          args:
            - '--default-authentication-plugin=mysql_native_password' # mysql 8만 해당 다른버전은 삭제할것
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: YOUR_PASSWORD
          ports:
            - containerPort: 3306
              name: mysql-slave02
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
            - name: config-volume
              mountPath: /etc/mysql/conf.d
      volumes:
        - name: mysql-volume
          persistentVolumeClaim:
            claimName: staging-slave02-pvc
        - name: config-volume
          configMap:
            name: slave02.cnf
```

slave03.yml

```yml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.beta.kubernetes.io/gid: '0'
  labels:
    type: local
  name: staging-slave03-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/staging/mysql-slave03
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  volumeMode: Filesystem

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    type: local
  name: staging-slave03-pvc
  namespace: staging
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      type: local
  storageClassName: manual
  volumeMode: Filesystem
  volumeName: staging-slave03-pv
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10G

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-slave03
  namespace: staging
  labels:
    app: auth
spec:
  selector:
    matchLabels:
      app: mysql-slave
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql-slave
    spec:
      nodeName: node14 #여기서 노드 할당
      containers:
        - image: mysql:8
          imagePullPolicy: Always
          name: mysql-slave03
          args:
            - '--default-authentication-plugin=mysql_native_password' # mysql 8만 해당 다른버전은 삭제할것
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: YOUR_PASSWORD
          ports:
            - containerPort: 3306
              name: mysql-slave03
          volumeMounts:
            - name: mysql-volume
              mountPath: /var/lib/mysql
            - name: config-volume
              mountPath: /etc/mysql/conf.d
      volumes:
        - name: mysql-volume
          persistentVolumeClaim:
            claimName: staging-slave03-pvc
        - name: config-volume
          configMap:
            name: slave03.cnf
```

### kuberentes에 배포하자.

```bash
kubectl apply -f configmap.yml
kubectl apply -f mysql-master.yml
kubectl apply -f mysql-slave01.yml
kubectl apply -f mysql-slave02.yml
kubectl apply -f mysql-slave03.yml
```

### 변수 선언

```bash
MASTER=`kubectl get pod | grep master | cut -d' ' -f1`
SLAVE01=`kubectl get pod | grep slave01 | cut -d' ' -f1`
SLAVE02=`kubectl get pod | grep slave02 | cut -d' ' -f1`
SLAVE03=`kubectl get pod | grep slave03 | cut -d' ' -f1`
echo $MASTER;
echo $SLAVE01;
echo $SLAVE02;
echo $SLAVE03;
```

### master 디비 백업

```bash
kubectl exec -it $MASTER -- mysqldump -u root -pYOUR_PASSWORD --all-databases > test.sql
```

mysqldump 버전에 따라 첫번째 줄에 이상한 내용이 들어가잇는경우가 있다. test.sql을 확인하여 지우자.

### master status 확인

```bash
kubectl exec -it $MASTER -- \
mysql -u root -pYOUR_PASSWORD <<EOF
show master status\G;
EOF
```

```sql
mysql> show master status\G
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 155
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)
```

이 정보를 나중에 사용할 것이다.

### slave 디비에 test.sql을 복구

```bash
kubectl exec -it $SLAVE01 -- mysql -u root -pYOUR_PASSWORD < test.sql
kubectl exec -it $SLAVE02 -- mysql -u root -pYOUR_PASSWORD < test.sql
kubectl exec -it $SLAVE03 -- mysql -u root -pYOUR_PASSWORD < test.sql
```

### slave setting

```bash
kubectl exec -it $SLAVE01 -- \
mysql -u root -pYOUR_PASSWORD <<EOF
CHANGE MASTER TO MASTER_HOST='mysql-master',
MASTER_USER='root',
MASTER_PASSWORD='YOUR_PASSWORD',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=105;
start slave;
show slave status\G;
EOF

kubectl exec -it $SLAVE02 -- \
mysql -u root -pYOUR_PASSWORD <<EOF
CHANGE MASTER TO MASTER_HOST='mysql-master',
MASTER_USER='root',
MASTER_PASSWORD='YOUR_PASSWORD',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=105;
start slave;
show slave status\G;
EOF

kubectl exec -it $SLAVE03 -- \
mysql -u root -pYOUR_PASSWORD <<EOF
CHANGE MASTER TO MASTER_HOST='mysql-master',
MASTER_USER='root',
MASTER_PASSWORD='YOUR_PASSWORD',
MASTER_LOG_FILE='mysql-bin.000003',
MASTER_LOG_POS=105;
start slave;
show slave status\G;
EOF
```

### 확인

마스터에서 데이터 입력

```bash
kubectl exec -it $MASTER -- \
mysql -u root -pYOUR_PASSWORD <<EOF
CREATE DATABASE test;
CREATE TABLE test.messages (message VARCHAR(250));
INSERT INTO test.messages VALUES ('hello');
EOF
```

slave에 데이터베이스 생긴것 확인

```bash
kubectl exec -it $SLAVE01 -- \
mysql -u root -pYOUR_PASSWORD <<EOF
SELECT * FROM test.messages;
EOF
```

slave service 동작 확인

### 서비스 동작 확인

다음 커맨드로 서비스 mysql-slave 로 호출을 해보면 랜덤하게 서버아이디를 받아오는것을 알수 있다. 로드 발란싱이 된다는것을 알수 있다.

```bash
kubectl run mysql-client-loop --image=mysql:8 -i -t --rm --restart=Never --\
  bash -ic "while sleep 1; do mysql -u root -pYOUR_PASSWORD -h mysql-slave -e 'SELECT @@server_id,NOW()'; done"
```

### slave상태 확인

```bash
kubectl exec -it $SLAVE02 -- \
mysql -u root -pYOUR_PASSWORD <<EOF
show slave status\G;
EOF
```

## delete all

```bash
kubectl delete -f configmap.yml
kubectl delete -f mysql-master.yml
kubectl delete -f mysql-slave01.yml
kubectl delete -f mysql-slave02.yml
kubectl delete -f mysql-slave03.yml

ssh node11 rm -rf /data/staging
ssh node12 rm -rf /data/staging
ssh node13 rm -rf /data/staging
ssh node14 rm -rf /data/staging
```
