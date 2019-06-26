---
layout: post
title: 'Kubernetes 에서 Gluster사용하기' 
author: teamsmiley
date: 2019-06-29
tags: [devops]
image: /files/covers/blog.jpg
category: {oauth}
---
# Kubernets에서 Gluster사용하기 - centos 7

중요 : namespace를 dev를 사용햇다. 각자 변경해서 사용하기 바란다.

기존에 추가해둔 gluster를 쿠버네티스에 Persistance Volume으로 사용해보려고 한다.

전체 쿠버네티스 클러스터에서 glusterfs가 동작해야한다. 설치한다. centos 7을 기준으로한다.

```bash
yum install -y glusterfs-cli-6.1-1.el7.x86_64
yum install -y glusterfs-client-xlators-6.1-1.el7.x86_64
yum install -y glusterfs-6.1-1.el7.x86_64
yum install -y glusterfs-fuse-6.1-1.el7.x86_64
yum install -y glusterfs-libs-6.1-1.el7.x86_64
```

* gluster 를 마운트해서 테스트해보자. 
```bash
mount -t glusterfs gluster12:gfs /glusterfs
cd /glusterfs 
ls
mkdir /glusterfs/kubernetes/mysql #아래에서 데이터를 넣을때 사용
```

* check uid/gid
```
ls -lnZ /mnt/glusterfs
> drwxr-xr-x 0 0 ?    ls 
```
위 커맨드에서 나온 uid/gid를 아래 에서 사용한다.
```
pv.beta.kubernetes.io/gid: "0" 
```

총 단계는 다음과 같다. 

endpoint생성 => pv 생성 => pvc생성 아래쪽부터 올라오면서 보면된다.
```
vi gluster-endpoint-pv-pvc.yaml
```

```yml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: glusterfs-pvc-dev01
  namespace: dev
  labels:
    type: local
spec:
  storageClassName: slow
  selector: 
    matchLabels: 
      type: local
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 30Gi

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-pv-dev01
  namespace: dev
  labels:
    type: local
  annotations:
    artifact.spinnaker.io/location: dev
    pv.beta.kubernetes.io/gid: "0" 
spec:
  storageClassName: slow 
  capacity:
    storage: 30Gi
  accessModes:
    - ReadWriteMany
  glusterfs:
    path: gfs
    endpoints: glusterfs
    readOnly: false
  persistentVolumeReclaimPolicy: Retain

---
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs
  namespace: dev
  labels:
    type: local
subsets: 
  - addresses:
      - ip: 192.168.0.196
    ports:
      - port: 1 
  - addresses:
      - ip: 192.168.0.197
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.198
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.199
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.200
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.201
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.202
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.203
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.204
    ports:
      - port: 1 
  - addresses:
      - ip: 192.168.0.205
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.206
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.207
    ports:
      - port: 1
  - addresses:
      - ip: 192.168.0.208
    ports:
      - port: 1
```

```
kubectl create -f gluster-endpoint-pv-pvc.yaml
```

subset은 전체 glusterip를 다 써주면 될듯 싶다. 

이제 glusterfs에 데이터를 쓸수 있다. 

확인
```bash
kubectl get pvc -n dev
kubectl get pv -n dev
kubectl get endpoints -n dev
```

이제 mysql을 실행해서 glusterfs에 데이터를 저장해보자.

```
vi mysql.yml
```
```yml
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
  namespace: dev
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value:
            password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: glusterfs
          mountPath: /var/lib/mysql
          subPath: kubernetes/mysql
      volumes:
      - name: glusterfs
        persistentVolumeClaim:
          claimName: glusterfs-pvc-dev01
```

```
kubectl create -f mysql.yaml
```

glusterfs의 루트에 마운트를 하고 subpath를 사용하여 하위특정 폴더에 mysql데이터를 저장해줬다.

이제 외부에 서비스를 오픈하면된다.
```
vi mysql-service.yaml
```

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: dev
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.82
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
```

```
kubectl create -f mysql-service.yaml
```

## 참고 
* namespace는 꼭 사용하기 바라고 이름은 바꿀수 있다.
* gid부분을 신경써서 처리하자.
* subpath 주의.

