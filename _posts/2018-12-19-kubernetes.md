---
layout: post
title: 'kubernetes 기본' 
author: teamsmiley
date: 2018-12-19
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes와 spinnaker를 설치 사용


## kubernetes 설치 

1개의 마스터 서버 (centos 7)
2개의 노드 서버 (centos 7)
1개의 미니오 스토리지 서버,registry (centos 7)

아이피는 다음과 같습니다. 

| | |
|---|---|
192.168.0.195 | master  |
192.168.0.192 | node192 |
192.168.0.193 | node193 |
192.168.0.194 | node194(minio storage, docker registry)| 
| | |

## 테스트랩 준비

centos7을 4대의 서버에 설치한다.

kubernetes를 설치하기전 해야할 일이 있습니다. 

<https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin>

문서를 참고하시면되는데요 진행해보겠습니다. 

저는 root로 로그인하여 진행하였습니다.

## before you begin kubernetes

```bash
# 기본 프로그램 설치
yum update && yum -y install kernel-headers kernel-devel 
yum install net-tools wget git -y # ifconfig git wget를 설치합니다. 

# selinux off
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# firewall off
systemctl disable firewalld

# docker install
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
systemctl start docker
systemctl enable docker

curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

modprobe br_netfilter
```

* host 파일 설정
```bash
vi /etc/hosts
> 192.168.0.195 master 
> 192.168.0.192 node192
> 192.168.0.193 node193
> 192.168.0.194 node194
```

* swap off
```bash
swapoff -a # 임시로 swap off 재부팅시 살아남.
vi /etc/fstab  # swap을 지우기 
```

## kubernetes package install

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl start kubelet
systemctl enable kubelet 
exit

exit
```
이렇게 하면 kubernetes설치까지 마무리 된것이다. 

node192 node193 을 위와 똑같이 만듭니다.(ansible을 사용하면 편리합니다.)


## node 194

* minio storage : daemon mode , spinnaker 에서 필요한 데이터를 저장하는 저장소 역할을 합니다. s3등을 사용하여도 되나 여기서는 vm에 하드디스크에 저장합니다. 
* docker registry : docker - docker private registry 2
* halyard : spinnaker installer - docker

```bash
mkdir -p /data/auth # registry 인증 정보 저장
mkdir -p /data/docker # docker-compose 위치
mkdir -p /data/registry # docker image 저장위치
mkdir -p /data/minio # minio file 저장 위치
mkdir -p /data/minio-config # minio 설정 저장위치 
mkdir -p /data/docker/registry # registry docker compose
mkdir -p /data/docker/halyard # halyard docker compose
mkdir -p /data/halyard # halyard 설정 저장위치
mkdir -p /data/kube # master config파일을 복사해둘 위치 halyard에서 이 파일을 참조해서 기본 설정을 만들기 때문
```

### minio 

```
cd
wget https://dl.minio.io/server/minio/release/linux-amd64/minio
chmod +x minio
mv minio /usr/local/bin/minio
minio server --address ":9001" --config-dir /data/minio-config /data/minio

Endpoint:  http://192.168.0.194:9001  http://172.17.0.1:9001  http://127.0.0.1:9001
AccessKey: OS2PVUL53ZTSNMJOWOWR
SecretKey: kLP1IdRqS+WqaLJ6WOXjrq80LptXy+j9SoeqXRLs
```

화면에 Access key와 secret key가 보인다 복사해두고 다음 커맨들를 사용하자. 

ctrl+c로 멈춘다. 

이제 데몬으로 띄워서 항상 동작하게 한다.

```bash
cat <<EOT >> /etc/default/minio
# Volume to be used for Minio server.
MINIO_VOLUMES="/data/minio/"
# Use if you want to run Minio on a custom port.
MINIO_OPTS="--address :9001"
# Access Key of the server.
MINIO_ACCESS_KEY=6K3MW29PYQHC4W39E03D
# Secret key of the server.
MINIO_SECRET_KEY=kuOkqn3y6UKvmHvC0DgoLyb+fDstDJFZV3NBwtZ1

EOT

# Download minio.service in /etc/systemd/system/
( cd /etc/systemd/system/; curl -O https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service )

vi /etc/systemd/system/minio.service # user와 그룹을 수정한다. 난 루트를 사용
User=root
Group=root

systemctl start minio
systemctl enable minio
```

http://localhost:9001 으로 확인한다. 파일도 넣어보고 폴더도 만들어보기 바란다.

### private repositry 실행 

<https://teamsmiley.github.io/2018/12/22/docker-private-registry/>

여기를 참고하셔도 됩니다. 

#### registry ssl 추가 
```bash
yum install epel-release python-pip  python-virtualenv -y
easy_install pip
pip install requests urllib3 pyOpenSSL --force --upgrade

cd /tmp
git clone https://github.com/certbot/certbot.git 
cd certbot

./certbot-auto certonly \
--manual \
--preferred-challenges=dns \
--email teamsmiley@gmail.com \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--debug \
--no-bootstrap \
-d registry.publishapi.com
```

_acme-challenge.registry txt 형태로 도메인에 등록요청

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.registry.publishapi.com with the following value:

r6hQID8GMkcW9ibu--vGVAZdENS01Qeu4GmfbV5OoMY

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

도메인관리 사이트에 접속하여 이 레코드를 추가합니다. 엔터는 아직 치시면 안됩니다.

저장하고 난후 조금 기다립니다. 그후 터미널로 들어와서 엔터를 칩니다. 

잘 생성되었습니다.

#### registry 유저 생성

```bash
docker run \
  --entrypoint htpasswd \
  registry -Bbn USERNAME PASSWORD > /data/auth/htpasswd
```

#### registry 실행
```
vi /data/docker/registry/docker-compose.yml
```

```yml
---
version: "3.3"
services:
  registry:
    container_name: 'registry'
    restart: always
    image: registry
    privileged: true
    ports:
      - 5000:5000
    environment:
      TZ: "America/Los_Angeles"
      REGISTRY_AUTH: htpasswd
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data/registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_HTTP_TLS_CERTIFICATE: /etc/letsencrypt/live/registry.publishapi.com/fullchain.pem
      REGISTRY_HTTP_TLS_KEY: /etc/letsencrypt/live/registry.publishapi.com/privkey.pem
    volumes:
      - /data/registry:/data/registry
      - /data/letsencrypt:/etc/letsencrypt
      - /data/auth:/auth
```


```bash
cd /data/docker/registry && docker-compose up -d
```



## init kubenetes - master

```bash
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
kubeadm init --apiserver-advertise-address 192.168.0.195 --pod-network-cidr 10.1.0.0/16
```
결과값을 잘 복사해두자. 나중에 이 값을 이용해서 노드를 마스터에 연결해준다.

```
You can now join any number of machines by running the following on each node
as root:

kubeadm join 192.168.0.195:6443 --token mcwwrn.12whl3ln7wxeoj2l --discovery-token-ca-cert-hash sha256:db16581f1259e99adbc2450a7e009835781aaa1a365d6abe762514075ac4485f
```

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

네트워크를 적용해야 하는데 weave를 적용해보자.

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
> serviceaccount/weave-net created
> clusterrole.rbac.authorization.k8s.io/weave-net created
> clusterrolebinding.rbac.authorization.k8s.io/weave-net created
> role.rbac.authorization.k8s.io/weave-net created
> rolebinding.rbac.authorization.k8s.io/weave-net created
> daemonset.extensions/weave-net created
```
네트워크관련 내용이 생성되었다.

이제 노드랑 연결을 해보자. 

## node를 마스터에 조인해자. 

위에 복사해둔 스크립트를 노드에 실행을 해준다.
```
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
echo '1' > /proc/sys/net/ipv4/ip_forward

kubeadm join 192.168.0.195:6443 --token mcwwrn.12whl3ln7wxeoj2l --discovery-token-ca-cert-hash sha256:db16581f1259e99adbc2450a7e009835781aaa1a365d6abe762514075ac4485f
```

## 클러스터 연결 확인
master 에서 노드가 붙었는지 확인할 필요가 있다.
```
$ kubectl get nodes
NAME      STATUS     ROLES    AGE     VERSION
master    Ready      master   6m51s   v1.13.0
node192   NotReady   <none>   15s     v1.13.0
node193   NotReady   <none>   12s     v1.13.0
```

계속확인하여 node가 ready상태로 변경될때가지 대기

not ready가 오래되면 다음처럼 한다. 

노드 상태를 확인해보자.

```bash
kubectl describe nodes node192
kubectl describe nodes node193
```

## kubenetes 기본 구조 공부
### namespace
### pod
### replicaset
### service
### deploy 

## kubectl 사용법

* get 
* describe 
* logs
* delete 
* apply 

```bash
kubectl get nodes
# default namespace의 pods를 보여줌 
kubectl get pods
# 전체 네임스페이스에 대해서 볼수 있음
kubectl get pods --all-namespaces    
kubectl get pods -o wide             # 더 많은 정보를 볼수있다. 어느노드에서 pod가 실행중인지 알수 있음
kubectl get pods --all-namespaces -w # w는 watch 계속 업데이트해준다. 
kubectl get pods hello-pod           # 특정 포드를 볼수 있음 

# pod의 로그를 보고싶은경우 
kubectl logs hello-pod

# replica set을 보고싶은경우
kubectl get replicaset

# services를 보고 싶은경우
kubectl get services
# deployment가 보고 싶은 경우
kubectl get deployment

# yml을 읽어서 pod service replicaset등을 쿠버네티스에 생성한다. 
kubectl create -f pod.yml
kubectl apply -f pod.yml

# pod의 자세한 설정으로 보고싶다. 
kubectl describe pods hello-pod

kubectl delete pods/hello-pod
kubectl delete replicationcontrollers/auth-server
kubectl delete svc auth-server-svc

# 추가 예제 
kubectl get all --all-namespaces

kubectl get pods --all-namespaces
kubectl get services --all-namespaces
kubectl get deploy --all-namespaces
kubectl get pv --all-namespaces
kubectl get pvc --all-namespaces
kubectl get rc --all-namespaces
kubectl get replicasets --all-namespaces
kubectl get secret --all-namespaces

kubectl delete pods --all
kubectl delete services --all
kubectl delete deploy --all
kubectl delete pv --all
kubectl delete pvc --all
kubectl delete rc --all
kubectl delete replicasets --all
kubectl delete secret --all
```

## yml 생성 (pod , service)를 만들어야한다. 마스터에서

```
cd /data/git/kube
vi hello-node.yml
```

```yml
---
apiVersion: v1
kind: Pod
metadata:
  name: hello-node
  labels:
    service-name: hello-node
spec:
  containers:
  - name: hello-node
    image: asbubam/hello-node
    readinessProbe:
      httpGet:
        path: /
        port: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080

---
apiVersion: v1
kind: Service # service생성
metadata:
  name: hello-node
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30100
  selector:
    service-name: hello-node
```
실습 해보자.

```bash
# kubectl get pods 명령으로 Pod list를 조회해보면 아직 아무것도 없다.
$ kubectl get pods
No resources found.

# kubectl create -f {yaml 파일명} 으로 Pod을 생성한다.
$ kubectl create -f hello-node.yml
pod "hello-node" created
service "hello-node" created

# kubectl get pods 결과에 1개의 Pod이 생성되고 있음을 확인할 수 있다.
$ kubectl get pods
NAME         READY     STATUS              RESTARTS   AGE
hello-node   0/1       ContainerCreating   0

# Pod이 정상적으로 실행되면 잠시 후에 아래와 같이 STATUS가 Running으로 update된다.
$ kubectl get pods
NAME         READY     STATUS    RESTARTS   AGE
hello-node   1/1       Running   0          13s

# pod의 자세한 내용을 본다.
$ kubectl describe pods hello-node

# localhost:8080에 접속해서 Hello World! 가 출력됨을 확인한다.
$  curl http://192.168.0.192:30100
Hello World!
```

## pod

하나의 pod에 여러개의 컨테이너를 넣을수 있다. pod안에서는 스토리지도 공유가 서로 가능하다. 도커컨테이너.
일반적으로는 하나의 컨테이너만 넣는듯. 

## 서비스

컨테이너는 클러스터 내에서 꺼지고 켜지고 하면서 아이피가 바뀌고 갯수가 바뀌기도 하고 한다. 그래서 컨테이너에 직접 접속하게 설정을 하면 언제 아이피가 바뀔지 모르니 안됨. 

서비스를 만드어서 외부에는 서비스를 오픈하고 서비스가 내부 pod를 연결해주는 구조로 되어있다. 

service type은 다음중 하나 고를수 있다. 

* NodePort
* ClusterIP
* LoadBalancer

위 예제에서는 nodeport로 오픈하고 있는것을 알수 있다. 

```
kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
hello-node   NodePort    10.103.218.93   <none>        8080:30001/TCP   6m9s   service-name=hello-node
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          38m    <none>
```

### node port
nodeport는 전체 노드에 특정 포트 (30000–32767)를 외부에 오픈한다.

그러므로 다음 두 커맨드는 모두 동작한다.
```
curl http://192.168.0.192:30100
curl http://192.168.0.193:30100
```

클러스터의 모든 서버 아이피에 포트를 연다.

외부에 오픈하기는 가장 쉬운 방법 - 단점도 있다 포트가 30000번이상, 2개의 다른 서비스에 하나의 포트를 사용할수 없다.  전체 클러스터에 포트를 열기 때문 

### ClusterIP
clusterip가 기본값이다. 처음에는 이걸 왜 쓰냐고 생각햇는데 내부 서비스끼리 연결될때 이걸 사용한다. 

예를 들면 워드프레스와 mysql 두개의 서비스가 돌고 잇을때 mysql은 외부에 오픈할 필요없이 워드프레스 pod에만 연결이 되면 되므로 mysql 서비스를 만들어서 cluster ip로 하면된다.

dns를 이용하여 내부적으로 이름만 가지면 아이피를 찾아서 연결해준다.

### Load Balance 

이것은 외부에서 아이피를 매핑해주는것으로 볼수 있다. 아마존이나 구글 클라우드는 자동으로 되는데 여기서는 베어메탈이므로 metallb라는 로드발란스를 설치를 해서 테스트해볼수 있다. 


## MetalLB 설치 

```bash
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml

# 로그를 잘 보기 바란다. 대충 이런것들이 생긴다 정도는 파악해야한다. 
> namespace/metallb-system created
> serviceaccount/controller created
> serviceaccount/speaker created
> clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
> clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
> role.rbac.authorization.k8s.io/config-watcher created
> clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
> clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
> rolebinding.rbac.authorization.k8s.io/config-watcher created
> daemonset.apps/speaker created
> deployment.apps/controller created
```

설정을 하자

vi metallb-ConfigMap.yml

```yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: my-ip-space
      protocol: layer2
      addresses:
      - 192.168.0.0/24
```
서비스에 줄 아이피를 정해뒀다 사용 가능한 아이피는 192.168.0.81 - 192.168.0.94 가 된다. 적용해보자.

```bash
kubectl create -f  metallb-ConfigMap.yml
# 상태를 확인해보자.
kubectl get pods -n metallb-system
kubectl logs -l component=speaker -n metallb-system
kubectl get svc -n metallb-system
```

이제 metallb-nginx.yml을 수정해서 loadbalance로 변경해보자.

```yml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1
        ports:
        - name: http
          containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

```bash
kubectl create -f metallb-nginx.yml
```

```bash
kubectl  get svc

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.106.215.59   <pending>     8080:30001/TCP   9s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          74m
```
type이 loadbalancer로 바귀었고 external-ip가 pending인것을 알수 있다. 

외부 아이피를 받을때까지 대기한다. ( 3분이 넘어가면 뭐가 잘못된것이므로 다시 해본다.)
```
$ kubectl  get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
hello-node   LoadBalancer   10.110.182.246   192.168.0.80   8080:30001/TCP   4s
kubernetes   ClusterIP      10.96.0.1        <none>         443/TCP          77m
```

외부 아이피를 잘 받았다. 이제 이 아이피를 호출해보면된다. 
```bash
curl http://192.168.0.80 
```

### metallb 삭제하고 싶으면 다음처럼 하면된다.
```
kubectl delete -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
kubectl delete -f metallb-ConfigMap.yml
kubectl get configmap -n metallb-system
kubectl get svc -n metallb-system
```

## kube dashboard 

### install

기본 템플릿을 다운받아서 로컬에서 서비스 타입을 로드발란스로 바꿔서 실행해보자.

```bash
vi kubernetes-dashboard.yaml
```

```yml
---
# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.81
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```

```
kubectl apply -f kubernetes-dashboard.yaml

> secret/kubernetes-dashboard-certs created
> serviceaccount/kubernetes-dashboard created
> role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
> rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
> deployment.apps/kubernetes-dashboard created
> service/kubernetes-dashboard created

kubectl get svc -n kube-system
> NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)         AGE
> kube-dns               ClusterIP      10.96.0.10      <none>         53/UDP,53/TCP   98m
> kubernetes-dashboard   LoadBalancer   10.107.42.182   192.168.0.81   443:32134/TCP   13s
```

외부 아이피 할당받았다 확인하자 

https://192.168.0.81

### 계정 추가 

vi kubernetes-dashboard-admin.yaml
```yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

적용하자
```bash
kubectl apply -f kubernetes-dashboard-admin.yaml
> serviceaccount/admin-user created
> clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

다음 명령을 이용해 Token을 알아내자.
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
토큰을 넣고 signin하면 로그인이 된다.

노드들에 cpu사용량들을 확인할수도 있으면 여러가지 편의 기능이 있다. 

네임스페이스 선택후 pod를 선택하면 상단 메뉴중 exec 라는것이 잇는데 이것을 누르면 컨테이너에 접속하여 shell을 실행할수 있는 웹사이트가 뜬다 . 가끔 임시로 쓰기는 좋아보임.

## mysql 서비스 설치하기 (namespace 꼭 사용하기)

kubernetes를사용할때는 꼭 namespace를 쓰기를 추천드립니다.
```bash
kubectl create namespace dev
kubectl create namespace prod
kubectl get namespaces
```

mysql은 pv,pvc생성 >> pod 생성 >> 서비스 생성 이런식으로 됩니다. 

### Create Local Persistent Volumes (on master)

```bash
mkdir -p /data/git/kube
cd /data/git/kube
vi dev-mysql-pv-pvc.yml
```
```yml
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: dev-mysql-pv-volume
  namespace: dev
  labels:
    type: local
spec:
  storageClassName: slow
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/dev-mysql"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node192 #호스트이름이 192번인 노드에 /data/dev-mysql이라고 만들어라.
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dev-mysql-pv-claim
  namespace: dev
spec:
  storageClassName: slow
  selector:
    matchLabels:
      type: local
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```
```bash
vi dev-mysql.yml
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
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```
```
kubectl create -f dev-mysql-pv-pvc.yml 
kubectl create -f dev-mysql.yml
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
```

Node192 번에 /data/dev-mysql폴더가 없으면 만들어 줘야한다. 

여기에 데이터를 저장하게 해두었으나 실제로는 nfs등에 저장하면될듯 

개발디비여서 외부에서 접속할 필요가 있으면 loadbalancer 를 서비스 타입으로 사용하나 서비스 디비는 외부에 오픈될 필요가 없으면 cluseterip를 사용해도 될듯 싶다. 


