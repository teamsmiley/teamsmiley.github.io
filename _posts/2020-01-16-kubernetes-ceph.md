---
layout: post
title: 'kubernetes ceph install' 
author: teamsmiley
date: 2020-01-16
tags: [kubernetes]
image: /files/covers/blog.jpg
category: {kuberentes}
---

node 4개가 있다. 그런데 노드에 하드가 4개씩 달려있다.

| | | | | | 
|-|-|-|-|-|
nodeName|sda|sdb|sdc|sdd|
node01|1T|1T|1T|1T|
node02|1T|1T|1T|1T|
node03|1T|1T|1T|1T|
node04|1T|1T|1T|1T|

sda는 os드라이브라 사용하지 않고 나머지 3개 하드 디스크를 ceph로 묵어서 스토리지로 쓰고 싶다. 

네트워크라 퍼포먼스 이슈가 있을수 있으니 잘 확인하고 쓴다. 

디비등은 문제가 잇을듯 보이고 파일서버나 image등을 http로 주는 서비스등은 잘 쓸수 있을듯 싶다. 

백업 용도로도 사용이 가능할듯 

# Rook with ceph

Rook를 쓰면 ceph를 쿠베 안에서 편하게 사용할수 있다.

## 설치전 주의사항
sdb sdc sdd 하드디스크가 fdisk가 되있으면 에러가 난다. 다음처럼 초기화를 하고 난후 위 과정을 실행해야한다.

```
wipefs -a /dev/sdb
wipefs -a /dev/sdc
wipefs -a /dev/sdd
```

## 설치 

```bash
git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml

kubectl -n rook-ceph get pod

kubectl apply -f toolbox.yaml
ns rook-ceph 

k get po | grep tool
> rook-ceph-tools-6586d4859f-cd4pb                   1/1     Running     0          21s

kubectl exec -it rook-ceph-tools-6586d4859f-cd4pb bash 
ceph status # 상태보여줌
ceph osd statu줌 # osd상태보여줌.
```


## dashboard
vi cluster.yaml에서 
```yml
spec:
  dashboard:
    enabled: true
```

```bash
kubectl apply -f cluster.yaml 
kubectl -n rook-ceph get service #확인

rook-ceph-mgr-dashboard    ClusterIP   10.106.86.124   <none>        8443/TCP            11m
```

vi dashboard-loadbalancer.yaml 
```
loadBalancerIP: 192.168.0.100
```
```bash
k apply -f dashboard-loadbalancer.yaml
kubectl -n rook-ceph get service
rook-ceph-mgr-dashboard-loadbalancer   LoadBalancer   10.104.37.219   192.168.0.100   8443:31518/TCP      5m53s
```
<https://192.168.0.100:8443> 으로 들어가면 볼수 있다.

크롬에서는 ssl validate 하면서 안될수도 있다. 파이어폭스나 다른 브라우저로 일단 해본다.

나중에 ssl없이 보기를 참고하자.

### 로그인 
id :  admin

password를 알아내자.

kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo

y1EUdqkEJ5

이렇게 패스워드를 알고 로그인하면 dash board를 볼수 있다.

### ssl없이 사용하기 
cluster.yaml 수정
```yml
  dashboard:
    enabled: true
    ssl: false #여기
```

dashboard-loadbalancer.yaml
```yml
spec:
  ports:
    - name: dashboard
      port: 7000 # ssl은 8443
      protocol: TCP
      targetPort: 7000 # ssl은 8443 
```

http://192.168.0.100:7000 된다.

host파일 변경하자

vi /etc/hosts 
```
192.168.0.105 ceph
``` 
http://ceph:7000 도 가능하게 한다.


## 하드나 노드들 추가.
cluster.yaml을 수정한후 `kubectl apply -f cluster.yaml` 을 한다.

전체 노드를 설정해두면 어떻게 되는지는 확인못햇음. 자동으로 추가할가?

## 사용






CephBlockPool 을 만들어서 사용한다. 다른 타입도 있다.

pool.yaml
```yml
#################################################################################################################
# Create a Ceph pool with settings for replication in production environments. A minimum of 3 OSDs on
# different hosts are required in this example.
#  kubectl create -f pool.yaml
#################################################################################################################

apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  # The failure domain will spread the replicas of the data across different failure zones
  failureDomain: host
  # For a pool based on raw copies, specify the number of copies. A size of 1 indicates no redundancy.
  replicated:
    size: 3
  # A key/value list of annotations
  annotations:
  #  key: valu
```

storage-class.yaml
```yml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  # clusterID is the namespace where the rook cluster is running
  clusterID: rook-ceph
  # Ceph pool into which the RBD image shall be created
  pool: replicapool

  # RBD image format. Defaults to "2".
  imageFormat: "2"

  # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
  imageFeatures: layering

  # The secrets contain Ceph admin credentials.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

  # Specify the filesystem type of the volume. If not specified, csi-provisioner
  # will set default as `ext4`.
  csi.storage.k8s.io/fstype: xfs

# Delete the rbd volume when a PVC is deleted
reclaimPolicy: Delete
```

```bash
k apply -f pool.yaml
k apply -f storage-class.yam
```

이제 pvc만 만들면 dynamic provisioning으로 pv를 만들어준다.

pod를 만들어서 마운트 해보면 
```bash
k apply -f pod.yml
k exec -it web-pod bash 
df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         426G   51G  353G  13% /
tmpfs            64M     0   64M   0% /dev
tmpfs            24G     0   24G   0% /sys/fs/cgroup
/dev/rbd0        60G   33M   60G   1% /mnt # 이것이 ceph
/dev/sda2       426G   51G  353G  13% /etc/hosts
shm              64M     0   64M   0% /dev/shm
tmpfs            24G   12K   24G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            24G     0   24G   0% /proc/acpi
tmpfs            24G     0   24G   0% /proc/scsi
tmpfs            24G     0   24G   0% /sys/firmware
```

주의할점은 StorageClass 이름으로 ceph에서 pvc에서 연결할수 있다.

```
cluster/examples/kubernetes
kubectl create -f mysql.yaml
```

mysql.yaml
```yml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block #위에서 만든 storageclassname
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: changeme
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim #위에 설정에서 정한 이름 
```
```bash
kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
mysql-pv-claim   Bound     pvc-95402dbc-efc0-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
```


## ceph storage type
1. block storage
1. object storage
1. file storage

위와 같은 타입이 있다.  여기에서 filesystem에 대해서 고민해보려고한다.

rook에서는 shared file system을 만들수 있다(1개만 가능) shared가 의미하는것은 Rook에서 만든  일반적인 file system 파드를 공유할수 없다. 1개의 파드에서 한개의 file system을 사용하게 된다. 그런데 이건 여러 파드에서 공유할수 있다. 그래서 그런지 클러스터당 1개만 가능하다.

filesystem.yaml
```yml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3
  dataPools:
    - replicated:
        size: 3
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```

```
k apply -f filesystem.yaml
kubectl -n rook-ceph get pod -l app=rook-ceph-mds
```

조금 기다리면 둘다 running이 된다.

Provision Storage

vi storage-class-filesystem.yml
```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cephfs
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # Root path of an existing CephFS volume
  # Required for provisionVolume: "false"
  # rootPath: /absolute/path

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

reclaimPolicy: Delete
```

k apply -f storage-class-filesystem.yml

이제 사용해볼가?

pvc만 만들어서 사용하면된다.



## error
1. osd가 안뜨고 osd-prepare만 뜨면 fswipe명령어 부분을 다시해본다.
1. ceph status
```
[errno 1] error connecting to the cluster
이러면 tool pod를 죽여보면 다시 시작하면서 붙는다. 
툴을 시작후 클러스터를 추가하면 이런 문제가 생기는듯.
```
1. rook 잘 안지워질때 
```
kubectl patch cephclusters.ceph.rook.io rook-ceph -p '{"metadata":{"finalizers":[]}}' --type=merge
```

