---
layout: post
title: 'kubernetes spinnaker 설치' 
author: teamsmiley
date: 2019-01-29
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# Spinnaker

## Install Halyard
Halyard manages the lifecycle of your Spinnaker deployment, including writing & validating your deployment’s configuration, deploying each of Spinnaker’s microservices, and updating the deployment.

로컬에 설치할수 있고 도커이미지를 사용할수 있다.

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
      - /data:/data
    ports:
      - 8084:8084
      - 9000:9000
```

```bash
cd /data/git/docker/halyard && docker-compose up -d

docker exec -it halyard bash
source <(hal --print-bash-completion) # 탭 완성 기능
exit
```

### Update Halyard on Docker
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

kubectl create -f spinnaker.yml

* halyard 에서 

First, make sure that the provider is enabled:
```
docker exec -it halyard bash
hal config provider kubernetes enable
```

Then add the account:
```bash
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --docker-registries my-registry --context $CONTEXT 

hal config features edit --artifacts true
```

### Choose your Environment 
다음 3가지를 지원한다. 
* Distributed installation on Kubernetes (추천)
* Local installations of Debian packages
* Local git installations from github

Distributed installation 을 사용할 것이다.

```bash
# Add Account
hal config deploy edit --type distributed --account-name my-k8s-v2-account
```

* docker private registry enable (on node194)
```bash
docker exec -it halyard bash
CONTEXT=$(kubectl config current-context)
hal config provider docker-registry enable
ADDRESS=registry.xgridcolo.com:5000 
REPOSITORIES="auth-server aaa bbb "
USERNAME=ragon #docker registry id/pass

hal config provider docker-registry account add my-registry \
    --repositories $REPOSITORIES \
    --address $ADDRESS \
    --username $USERNAME \
    --password 
# password는 프롬프트로 물어본다.

hal deploy apply
```

* 나중에 registry 추가 

```bash
docker exec -it halyard bash
hal config provider docker-registry account list
hal config provider docker-registry account  edit  my-registry --repositories="aaa,bbb"
hal deploy apply
```

### Choose a storage service
다음과 같은 스토리지를 사용할수 있다.
* Azure Storage
* Google Cloud Storage
* Minio
* Redis :warning: Unsupported and not recommended for production environments
* S3
* Oracle Object Storage

우리는 minio를 사용한다. 

#### minio install 
<https://teamsmiley.github.io/2019/01/29/minio/> 여기 참고해서 설치 가능하다.

#### halyard minio 설정

halyard 컨테이너에서 다음 실행

```bash
docker exec -it halyard bash

MINIO_ACCESS_KEY=6K3MW29PYQHC4W39E03D
MINIO_SECRET_KEY=kuOkqn3y6UKvmHvC0DgoLyb+fDstDJFZV3NBwtZ1
ENDPOINT=http://192.168.0.194:9001

echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key 

# hal config storage edit --type s3
hal deploy apply
```
s3로 하는 이유는 아마존과는 상관이 없고 minio가 s3와 compatible 하기 때문이다.

그러나 versioning은 지원하지 않아서 설정을 변경해야한다.

node194에서

```bash
mkdir -p ~/.hal/default/profiles/
vi ~/.hal/default/profiles/front50-local.yml
> spinnaker.s3.versioning: false
```

### Deploy Spinnaker and Connect to the UI 

컨테이너로 접속해서 
```bash
docker exec -it halyard bash
hal version list #사용할 버전을 고른다. 난 1.11.x

hal config version edit --version 1.11.x

hal deploy apply
```

halyard가 쿠베에 접속해서 디플로이를 순서대로 한다. 

### Connect to the Spinnaker UI - 화면을 보자. (master에서 )

```
kubectl get svc -n spinnaker
```

화면을 보는 방법은 여러가지가 있다.
1. ssh tunneling을 이용해서 보는 방법 - spinnaker는 기본으로 이것만 사용할수 있게 되있다.
2. 서비스 타입을 바꿔서 외부에 오픈하는 방법 spin-deck 와 spin-gate를 LoadBalancer나 NodePort로  바꿔 주면 된다.
3. Ingress 를 하나 만들어서 백앤드 서비스로 spin-deck 와 spin-gate를 붙여주면 된다. 

3번으로 진행

그럼 인그레스를 이용해서 해보자. 

### ingress 이용 (on master)

vi spinnaker-ingress.yml

```yml
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

---
apiVersion: v1
kind: Service
metadata:
  name: spinnaker-ingress
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
```

```
kubectl create -f spinnaker-ingress.yml
```

hosts파일에 설정을 하자. (on laptop)

vi /etc/hosts
```
204.16.116.86 spin.publishapi.com 
204.16.116.86 spin-gate.publishapi.com
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

인그레스 서비스를 사용하여 바꿔서 외부로 오픈하고 외부에서 사용하는 url을  override-base-url을 써서 설정을 바꿔준후  hal deploy apply를 하면 클러스터로 설정이 넘어간다. 

그 후 웹브라우저로 접속해보면 된다.

## spin-front50 crash 발생
```bash
$ kubectl get pods -n spinnaker
NAME                                READY   STATUS             RESTARTS   AGE
spin-front50-7cf9844bcb-hf8fg       0/1     CrashLoopBackOff   15         61m
spin-front50-fcdbb667c-fk8kz        0/1     CrashLoopBackOff   18         74m
```

minio server실행이 되있는지 확인한다.

## blue green 배포 적용 

<https://www.spinnaker.io/guides/user/kubernetes-v2/traffic-management/#sample-bluegreen-pipeline>

여기를 잘 참고하면 된다. 

영어가 불편하신분을 위해 설명을 하면  

1. application을 만들자.  
2. load balancer를 만들자. (service)
3. pipeline을 추가한다. 

아직은 pv pvc는 생성이 안되는듯 보여 직접 kubectl로 적용했고 ingress도 생성이 안되는듯 보여 수동으로 생성했다. (방법 아시는분은 댓글좀..)

해보자.

### 서비스를 만들자. (load balancer)

```yml
kind: Service
  apiVersion: v1
  metadata:
    name: my-service
  spec:
    selector:
      app: myapp
    ports:
    - protocol: TCP
      port: 80
```

### 파이프 라인을 추가하자. 

1. 트리거를 추가하자. 
```
configurtion ==> 트리거 적용 >> docker registry  >> name >> image >> enable trigger
```
태그가 번호가 꼭 바뀌어야 이 트리거가 실행된다.

2. 스테이지 추가 

add stage >> Deploy(Manifest) >> text 

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    strategy.spinnaker.io/max-version-history: '2' # 최대 보관 이미지 갯수 롤백이 최근 1번째까지 가능하게 된다.
    traffic.spinnaker.io/load-balancers: '["service my-service"]' #위에 만들어둔 서비스 붙인다. 이게 없으면 disable이 안된다. 
  labels:
    tier: frontend
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - image: 'UR-REGISTRY:5000/UR-IMAGE:${trigger["tag"]}' # 이거 중요 트리거에서 넘겨준 정보를 가지고 빌드한다.
          name: frontend
```

${trigger["tag"]} 이 부분이 트리거에서 넘겨주는 값을 가지고 빌드를 하는 부분

파이프라인을 빌드하면 서비스가 생성되고 기존 서비스는 disable된다. 

2개 이상의 pod들은 전부 삭제된다.

### add disable(manifest) 
add stage >> disable(manifest) >> edit json
```
{
  "account": "my-k8s-v2-account",
  "app": "renderfarm",
  "cloudProvider": "kubernetes",
  "cluster": "replicaSet renderingfarm-frontend",
  "criteria": "second_newest",
  "kind": "replicaSet",
  "location": "publish-api-live",
  "manifestName": null,
  "mode": "dynamic",
  "name": "Disable (Manifest)",
  "type": "disableManifest"
}
```
