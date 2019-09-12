---
layout: post
title: 'spinnaker 설치' 
author: teamsmiley
date: 2019-01-29
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes,spinnaker}
---

# Spinnaker 설치 

## Install Halyard
Halyard manages the lifecycle of your Spinnaker deployment, including writing & validating your deployment’s configuration, deploying each of Spinnaker’s microservices, and updating the deployment.

로컬에 설치할수 있고 도커이미지를 사용할수 있다. 도커를 사용한다.

```bash
mkdir ~/.hal
mkdir /data/git/docker/halyard/kube
vi /data/git/docker/halyard/docker-compose.yml
```

```yml
---
version: "3.3"
services:
  halyard:
    container_name: halyard
    restart: always
    image: gcr.io/spinnaker-marketplace/halyard:stable
    volumes:
      - ./kube:/home/spinnaker/.kube
    ports:
      - 8084:8084
      - 9000:9000
```

```bash
cd /data/git/docker/halyard && docker-compose up -d

docker exec -it halyard bash

# 탭 완성 기능
source <(hal --print-bash-completion)
hal version list
hal config version edit --version 1.15.3

# setup storage : minio 
MINIO_ACCESS_KEY=6K3MW29PYQHC4W39E03D
MINIO_SECRET_KEY=kuOkqn3y6UKvmHvC0DgoLyb+fDstDJFZV3NBwtZ1
ENDPOINT=http://192.168.0.194:9001

echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key 

hal config storage edit --type s3 #minio가 s3와 compatible 하기 때문이다. 그러나 versioning은 지원하지 않아서 설정을 변경해야한다.
mkdir -p ~/.hal/default/profiles
echo 'spinnaker.s3.versioning: false' > ~/.hal/default/profiles/front50-local.yml

# kubernets 지원
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --context $CONTEXT

hal config features edit --artifacts true

hal config deploy edit --type distributed --account-name my-k8s-v2-account

# docker registery
hal config provider docker-registry enable

ACCOUNT=my-registry
ADDRESS=registry.xgridcolo.com:5000
REPOSITORIES="api,www"
USERNAME=XXX

hal config provider docker-registry account add $ACCOUNT\
    --provider-version v2 \
    --repositories $REPOSITORIES \
    --address $ADDRESS \
    --username $USERNAME \
    --password 

## spinnaker base url 
hal config security ui edit --override-base-url http://spin.publishapi.com 
hal config security api edit --override-base-url http://spin-gate.publishapi.com

hal deploy apply
```

## Update Halyard on Docker
```bash
docker pull gcr.io/spinnaker-marketplace/halyard:stable
cd /data/git/docker/halyard && docker-compose down && docker-compose up -d
```

## Choose Cloud Providers

스피네커는 여러 프로바이더를 제공한다. 아마존 또는 쿠버네티스 또는 오픈 스택등에 사용할수 있다.

전체 리스트는 다음 페이지에서 확인한다.

<https://www.spinnaker.io/setup/install/providers/>

우리는 Kubernetes V2 (manifest based) 를 사용한다.

쿠버네티스 마스터 노드에 config를 복사하여 halyard 도커에서 참조할수 있도록 컨테이너에 넣어줘야한다.

```
scp master:~/.kube/config /data/git/docker/halyard/kube/
```

docker-compose파일이 kube 볼륨 매핑을 하고 있어 컨테이너가 이 파일을 참조할수 있다. 

### Adding an account (halyard container)

* kubernetes에 RBAC 추가  (on master)

```bash
kubectl create namespace spinnaker
vi /data/git/kube/spinnaker-rabc.yml
```
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: spinnaker-role
rules:
- apiGroups: [""]
  resources: ["namespaces", "configmaps", "events", "replicationcontrollers", "serviceaccounts", "pods/log"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods", "services", "secrets"]
  verbs: ["create", "delete", "deletecollection", "get", "list", "patch", "update", "watch"]
- apiGroups: ["autoscaling"]
  resources: ["horizontalpodautoscalers"]
  verbs: ["list", "get"]
- apiGroups: ["apps"]
  resources: ["controllerrevisions", "statefulsets"]
  verbs: ["list"]
- apiGroups: ["extensions", "apps"]
  resources: ["deployments", "replicasets", "ingresses"]
  verbs: ["create", "delete", "deletecollection", "get", "list", "patch", "update", "watch"]
# These permissions are necessary for halyard to operate. We use this role also to deploy Spinnaker itself.
- apiGroups: [""]
  resources: ["services/proxy", "pods/portforward"]
  verbs: ["create", "delete", "deletecollection", "get", "list", "patch", "update", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: spinnaker-role-binding
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: spinnaker-role
subjects:
- namespace: spinnaker
  kind: ServiceAccount
  name: spinnaker-service-account
---
apiVersion: v1
kind: ServiceAccount
metadata:
 name: spinnaker-service-account
 namespace: spinnaker
```

kubectl create -f spinnaker-rabc.yml

### registry 추가/삭제
```
hal config provider docker-registry account edit $ACCOUNT --add-repository new-api
hal config provider docker-registry account edit $ACCOUNT --remove-repository old-api
hal deploy apply
```

## Choose a storage service
다음과 같은 스토리지를 사용할수 있다.
* Azure Storage
* Google Cloud Storage
* Minio
* Redis :warning: Unsupported and not recommended for production environments
* S3
* Oracle Object Storage

우리는 minio를 사용한다. 

## minio install 
<https://teamsmiley.github.io/2019/01/28/minio/> 여기 참고해서 설치 가능하다.

halyard가 쿠베에 접속해서 디플로이를 순서대로 한다. 

## Connect to the Spinnaker UI - 화면을 보자. (master에서 )
```
kubectl get svc -n spinnaker
```

Ingress 를 하나 만들어서 백앤드 서비스로 spin-deck 와 spin-gate를 붙여주면 된다. 

인그레스 서비스를 사용하여 바꿔서 외부로 오픈하고 외부에서 사용하는 url을  override-base-url을 써서 설정을 바꿔준후  hal deploy apply를 하면 클러스터로 설정이 넘어간다. 

그 후 웹브라우저로 접속해보면 된다.

### ingress 이용 (on master)

vi spinnaker-ingress.yml

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-spin
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.84
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-spin
  namespace: spinnaker
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: spin.publishapi.com
    http:
      paths:
      - backend:
          serviceName: spin-deck
          servicePort: 9000

  - host: spin-gate.publishapi.com
    http:
      paths:
      - backend:
          serviceName: spin-gate
          servicePort: 8084
```

```
kubectl apply -f spinnaker-ingress.yml
```

hosts파일에 설정을 하자. (on laptop)

vi /etc/hosts
```
204.16.116.84 spin.publishapi.com 
204.16.116.84 spin-gate.publishapi.com
```

halyard에서 설정을 업데이트해서 클러스터로 밀어 넣어준다. 

```bash
docker exec -it halyard bash

hal config security ui edit --override-base-url http://spin.publishapi.com 
hal config security api edit --override-base-url http://spin-gate.publishapi.com
hal deploy apply
```

http://spin.publishapi.com

드디어 화면이 보인다.

## spin-front50 crash

```bash
$ kubectl get pods -n spinnaker
NAME                                READY   STATUS             RESTARTS   AGE
spin-front50-7cf9844bcb-hf8fg       0/1     CrashLoopBackOff   15         61m
spin-front50-fcdbb667c-fk8kz        0/1     CrashLoopBackOff   18         74m
```
minio server 실행이 되었는지 정보(엑세스키 시크릿키) 정확한지 확인한다.

## halyard backup and restore

halyard를 설정을 하면 일단 로컬에  config파일에 정보를 저장한다. 그러므로 도커이미지가 재시작되면 이 정보는 모두 사라져있게 된다. 

그러므로 백업을 항상 받아둬야한다.

```bash
ssh node194
docker exec -it halyard bash 
hal backup create
exit
# 생성 완료 되었으므로 컨테이너 외부로 빼둔다.
FILENAME=halyard-2019-08-27_05-26-14-550Z.tar
docker cp halyard:/home/spinnaker/$FILENAME /glusterfs/backup/k8s/halyard/

#정리 스크립트 
ssh minio
docker exec halyard bash -c "hal backup create" 
docker exec halyard bash -c "mv /home/spinnaker/*.tar /backup" 
mv /data/git/docker/halyard/backup/*.tar /glusterfs/backup/k8s/halyard

docker-compose down && docker-compose up -d 

# 설정이 없어진것을 확인해라 
ls ~/.hal

# 복구하자.
ssh minio
docker-compose up -d 
FILENAME=halyard-2019-08-27_05-26-14-550Z.tar
cp /glusterfs/backup/k8s/halyard/$FILENAME /data/git/docker/halyard/backup

docker exec -it halyard bash 
hal backup restore --backup-path /backup/$FILENAME

> This will override your entire current halconfig directory. Do you want to continue? (y/N) y
> + Restore backup
>   Success
> + Successfully restored your backup.

# 확인
hal config provider docker-registry -o yaml
```

그러므로 항상 백업을 해주어야한다.

기본 설정 파일과 다르게 웹사이트에서 설정한 데이터는 위에서 설정한 스토리지에 (미니오) 전부 저장이 된다. 이것도 별도로 백업을 해둬야할듯 




