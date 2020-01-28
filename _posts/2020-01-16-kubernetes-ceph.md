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
```
kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo

> l1EUdqkEJ5
```

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


## 하드나 노드들 추가

cluster.yaml을 수정한후 `kubectl apply -f cluster.yaml` 을 한다.

전체 노드를 설정해두면 어떻게 되는지는 확인못햇음. 자동으로 추가할가?

## 개요

Rook에서는 다음을 사용이 가능합니다.

* Ceph Storage
* EdgeFS Data Fabric
* Cassandra Cluster CRD
* CockroachDB Cluster CRD
* Minio Object Store CRD
* NFS Server CRD
* YugabyteDB Cluster CRD

CRD는 쿠버네티스에서 custom resource definition 을 말합니다. 

위 내용을 다 확인하면 좋겟으나 나는 ceph를 쓴다. 그것만 해보자.

ceph는 또 다음과 같은 구성내용을 되어있다.

* Block Storage
* Object Storage
* Shared Filesystem

이 내용은 꼭 공부들 해서 뭔의미인지 파악해볼것.

그리고 다음 CRD를 지원한다.
* Cluster CRD
* Block Pool CRD
* Object Store CRD
* Object Bucket Claim
* Object Store User CRD
* Shared Filesystem CRD
* NFS CRD
* Ceph CSI
* Client CRD

각각 crd에 대해서 공부해서 알아둬야한다.

## storage 종류

* file system : 일반적인 파일시스템 폴더로 구분된다. nas에서 사용, 

* object storage : 파일로 저장은되나 key로 구분이 된다. key 에 파일이 매핑되서 보여진다. 기본적으로 수정은 불가능하다. 삭제후 새로 생성은 가능.  key를 보면 폴더처럼 보이게 하위구조로 만들어 두어서 기존 파일시스템처럼 보일수는 있으나 이 구조자체가 key라고 봐야한다. s3등. restapi를 통해서 처리한다.

* block storage : 위 두개 스토리지는 파일로 저장을 하나 이건 블록단위로 나눠서 파일을 저장후 요청이 들어오면 그 블록들을 합쳐서 보내주는것이라고 함. SAN을 주로 이걸로 사용함.

## ceph storage type
ceph는 위 3가지를 다 지원을 한다. 

* Block Storage
* Object Storage
  * s3를 사용하던 클라이언트는 다 지원 가능
  * 스토리지에 저장(s3에 저장되는지 알앗으나 아니였음)
* Shared Filesystem (클러스터당 1개만 생성 가능) 
  * 여러 파드에 동시에 붙어질수 있다.
  * rook에서는 shared file system을 만들수 있다(1개만 가능) 
  * shared가 의미하는것은 Rook에서 만든  일반적인 file system 파드를 공유할수 없다. 
  * 1개의 파드에서 한개의 file system을 사용하게 된다. 그런데 이건 여러 파드에서 공유할수 있다. 그래서 그런지 클러스터당 1개만 가능하다.
  * By default only one shared filesystem can be created with Rook. Multiple filesystem support in Ceph is still considered experimental and can be enabled with the environment variable ROOK_ALLOW_MULTIPLE_FILESYSTEMS defined in operator.yaml.

## CRD (Custom Resource Definition)

CRD는 다음과 같은 구분이 있다.

* Cluster CRD
* Block Pool CRD
* Object Store CRD
* Object Bucket Claim
* Object Store User CRD
* Shared Filesystem CRD
* NFS CRD
* Ceph CSI
* Client CRD

### Cluster CRD
1. Specify host paths and raw devices
1. Specify the storage class Rook should use to consume storage via PVCs

* Host-based Cluster
```yml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    # see the "Cluster Settings" section below for more details on which image of ceph to run
    image: ceph/ceph:v14.2.6
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
    allowMultiplePerNode: true
  storage:
    useAllNodes: true
    useAllDevices: true
```

* PVC-based Cluster
```yml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
    volumeClaimTemplate:
      spec:
        storageClassName: local-storage
        resources:
          requests:
            storage: 10Gi
  storage:
   storageClassDeviceSets:
    - name: set1
      count: 3
      portable: false
      tuneSlowDeviceClass: false
      volumeClaimTemplates:
      - metadata:
          name: data
        spec:
          resources:
            requests:
              storage: 10Gi
          # IMPORTANT: Change the storage class depending on your environment (e.g. local-storage, gp2)
          storageClassName: local-storage
          volumeMode: Block
          accessModes:
            - ReadWriteOnce
```

### Block Pool CRD
```yml
apiVersion: ceph.rook.io/v1
kind: CephBlockPoo #여기 주의
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
  deviceClass: hdd
```

### Object Store CRD

create CephObjectStore ==>  create storageClass ==> create bucket 

#### CephObjectStore
```yml
apiVersion: ceph.rook.io/v1
kind: CephObjectStore #여기 주의
metadata:
  name: my-store
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: host
    replicated:
      size: 3
  dataPool:
    failureDomain: host
    erasureCoded:
      dataChunks: 2
      codingChunks: 1
  preservePoolsOnDelete: true
  gateway:
    type: s3
    sslCertificateRef:
    port: 80
    securePort:
    instances: 1
```

#### StorageClass and Object Bucket Claim

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-bucket
provisioner: ceph.rook.io/bucket
reclaimPolicy: Delete
parameters:
  objectStoreName: my-store
  objectStoreNamespace: rook-ceph
  region: us-east-1
---
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: ceph-bucket
spec:
  bucketName: ceph-bucket # 이름으로 하나생성
  # generateBucketName: ceph-bucket  # 이름 뒤에 복잡한 문자를 붙여서 자동 생성 BucketName을 같이 주면 BucketName 사용 
  storageClassName: rook-ceph-bucket
```

#### Client Connections
```bash
#config-map, secret, OBC will part of default if no specific name space mentioned
export AWS_HOST=$(kubectl get cm ceph-bucket -o yaml | grep BUCKET_HOST | awk '{print $2}')
export AWS_ACCESS_KEY_ID=$(kubectl get secret ceph-bucket -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | base64 --decode)
export AWS_SECRET_ACCESS_KEY=$(kubectl get secret ceph-bucket -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | base64 --decode)

echo $AWS_HOST
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
```

이것들을 적어두고 나중에 사용한다.

rook-ceph-rgw-my-object-storage.rook-ceph
8G3C6FR2X7CCRAM9IJ5D
zoFd26tPCiGjWe0XP37lGJJllPvm7xO4OCVmqGCQ

#### 사용
Consume the Object Storage

* AWS_ENDPOINT
`kubectl get svc rook-ceph-rgw-my-object-storage`
```
rook-ceph-rgw-my-object-storage        ClusterIP      10.97.24.211    <none>          80/TCP              21m
```

`k describe ObjectBucketClaim ceph-bucket | grep "Bucket Name" `
```
Bucket Name:           ceph-bucket-8e636b8f-d85e-4799-95b0-d2a209dbac6d
```
tool에서 해보자.
```bash
export AWS_HOST=rook-ceph-rgw-my-object-storage.rook-ceph
#export AWS_ENDPOINT=10.97.24.211:80
export AWS_ACCESS_KEY_ID=8G3C6FR2X7CCRAM9IJ5D
export AWS_SECRET_ACCESS_KEY=zoFd26tPCiGjWe0XP37lGJJllPvm7xO4OCVmqGCQ
echo $AWS_HOST
#echo $AWS_ENDPOINT 
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
```

버킷을 생성하지 않으면 위 정보가 없으므로 아래처럼 user를 만들어서 해야한다.

테스트하기 위해 다음 패키지가 필요

```bash
yum --assumeyes install s3cmd
```

```bash
echo "Hello Rook2" > /tmp/rookObj
s3cmd put /tmp/rookObj --no-ssl --host=${AWS_HOST} --host-bucket= s3://ceph-bucket-8e636b8f-d85e-4799-95b0-d2a209dbac6d
s3cmd get s3://ceph-bucket-8e636b8f-d85e-4799-95b0-d2a209dbac6d/rookObj /tmp/rookObj-download --no-ssl --host=${AWS_HOST} --host-bucket=
cat /tmp/rookObj-download
s3cmd ls s3://ceph-bucket-8e636b8f-d85e-4799-95b0-d2a209dbac6d /tmp/rookObj-download --no-ssl --host=${AWS_HOST} --host-bucket=
```

`--host-bucket=  ` 다음에 공백이 중요 없으면 에러

AWS_ENDPOINT는 필요가 없는것도 같음.

#### 외부에서 연결

```
k get svc rook-ceph-rgw-my-object-storage --show-labels

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   LABELS
rook-ceph-rgw-my-object-storage   ClusterIP   10.97.24.211   <none>        80/TCP    49m   app=rook-ceph-rgw,ceph_daemon_id=my-object-storage,rgw=my-object-storage,rook_cluster=rook-ceph,rook_object_store=my-object-storage
```
```yml
# 외부에 오픈
---
apiVersion: v1
kind: Service
metadata:
  name: rook-ceph-rgw-my-store-external
  namespace: rook-ceph
  labels:
    app: rook-ceph-rgw
    rook_cluster: rook-ceph
    rook_object_store: my-object-storage
spec:
  ports:
    - name: rgw
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: rook-ceph-rgw
    rook_cluster: rook-ceph
    rook_object_store: my-object-storage
  sessionAffinity: None
  type: LoadBalancer
  loadBalancerIP: 192.168.0.198
```

k apply -f object-storage

이제 laptop에서 위 커맨드를 실행해서 로그인해본다.
```bash
export AWS_HOST=192.168.0.198
export AWS_ACCESS_KEY_ID=8G3C6FR2X7CCRAM9IJ5D
export AWS_SECRET_ACCESS_KEY=zoFd26tPCiGjWe0XP37lGJJllPvm7xO4OCVmqGCQ

brew install s3cmd
s3cmd get s3://ceph-bucket-8e636b8f-d85e-4799-95b0-d2a209dbac6d/rookObj /tmp/rookObj-download --no-ssl --host=${AWS_HOST} --host-bucket=
```

파일을 잘 가져온다.


#### Object Store User CRD
```yml
apiVersion: ceph.rook.io/v1
kind: CephObjectStoreUser # 여기 주의
metadata:
  name: my-user
  namespace: rook-ceph
spec:
  store: my-store
  displayName: my-display-name 
```

kubectl create -f object-user.yaml

kubectl -n rook-ceph describe secret rook-ceph-object-user-my-store-my-user

이걸 pod에 마운트해서 사용하면 된다. 고 메뉴얼에 써져잇는데 이걸 모르겟네 //todo

비번을 직접 찾아보려면 

kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o yaml | grep AccessKey | awk '{print $2}' | base64 --decode
kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o yaml | grep SecretKey | awk '{print $2}' | base64 --decode

### Shared Filesystem CRD
```yml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: host
    replicated:
      size: 3
  dataPools:
    - failureDomain: host
      replicated:
        size: 3
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```

### NFS CRD 
* nfs-ganesha를 사용해서 nfs를 지원 
* nfs share of the filesystem or object store through the CephNFS custom resource definition.

```yml
apiVersion: ceph.rook.io/v1
kind: CephNFS
metadata:
  name: my-nfs
  namespace: rook-ceph
spec:
  rados:
    pool: myfs-data0
    namespace: nfs-ns
  server:
    # the number of active NFS servers
    active: 2
```

### Ceph CSI 
  There are two CSI drivers integrated with Rook
  * RBD: This driver is optimized for RWO pod access where only one pod may access the storage
  * CephFS: This driver allows for RWX with one or more pods accessing the same storage
  
  RBD Snapshots : enable the VolumeSnapshotDataSource feature gate on your Kubernetes cluster API server.
  `--feature-gates=VolumeSnapshotDataSource=true`

### Ceph Client CRD
  * Don’t use Client CRD for Flex or CSI driver users, they are created automatically.
  * ceph client를 이용해서 처리하는듯.


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

### block storage
single pod가 mount 된다.

`vi block-storage-ceph-block-pool.yaml`

```yml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    clusterID: rook-ceph
    pool: replicapool
    imageFormat: "2"
    imageFeatures: layering
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    csi.storage.k8s.io/fstype: xfs
reclaimPolicy: Delete
```

`kubectl create -f block-storage-ceph-block-pool.yaml`

조금 기다리면 자동으로 생긴다.

* pod를 연결해보자.

k apply -f mysql.yml

k exec -it mysql bash

df -h
```bash
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

cp -R /etc/ /mnt/ 

io를 만들어보자.

dash board로 확인하면 데이터가 움직이는게 보닌다.

pod에 마운트해서 사용할수가 있다. 
### object storage 

### shared file storage
multiple pod가 마운트 된다.

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
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  clusterID: rook-ceph
  fsName: myfs
  pool: myfs-data0
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

