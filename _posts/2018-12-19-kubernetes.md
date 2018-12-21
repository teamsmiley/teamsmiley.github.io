---
layout: post
title: '초보 Devops - kubernetes' 
author: teamsmiley
date: 2018-12-19
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# 테스트 랩 만들기 

homebrew 

home brew cask 

homebrew install https://brew.sh/index_ko


Homebrew-Cask은 Google 크롬 또는 Atom과 같은 GUI 응용 프로그램을 설치하는 Homebrew의 확장 프로그램입니다. 독립적으로 시작되었지만 유지 보수 담당자는 Homebrew의 핵심 팀과 긴밀하게 협력합니다.

brew install brew-cask
brew cask install virtualbox
brew cask install vagrant

## virtualbox install

## vagrant install

# kubernetes 

구조 총 3개의 서버 (centos 7)

192.168.0.191 master
192.168.0.192 node192
192.168.0.193 node193

## 설치 - Master,Node192,Node193

### 기본 서버 설정
<https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin>

```bash
hostnamectl set-hostname 'master'
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# firewall off
systemctl disable firewalld

# docker install
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
systemctl start docker
systemctl enable docker

modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

* host 파일 설정
```
vi /etc/hosts
```
```
192.168.0.191 master
192.168.0.192 node192
192.168.0.193 node193
```

* 스왑오프
```bash
swapoff -a # 임시로 스왑 오프 재부팅시 살아남.
vi /etc/fstab  # swap을 지우기 
```

### kubernetes package install 

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

systemctl enable kubelet && systemctl start kubelet
```

## init kubenetes - Master

```bash
kubeadm init
# 또는 
kubeadm init --pod-network-cidr=192.168.0.0/24
```
결과값을 잘 복사해두자. 나중에 이 값을 이용해서 노드를 마스터에 연결해준다.

```
You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.0.191:6443 --token lr98l5.962xm2vi5pznrdhz --discovery-token-ca-cert-hash sha256:d1ffcec6e71cc2be3105adf450d80f179462db35abe47155820bab852ce1d6f5
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
kubeadm join 192.168.0.191:6443 --token lr98l5.962xm2vi5pznrdhz --discovery-token-ca-cert-hash sha256:d1ffcec6e71cc2be3105adf450d80f179462db35abe47155820bab852ce1d6f5
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

노드 상태를 확인해보자.

```bash
kubectl describe nodes master
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

# 추가 샘플 
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

## yml 생성 (pod , service)를 만들어야한다.

vi hello-node.yml

```yml
---
apiVersion: v1
kind: Pod  # pod 생성 
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
    nodePort: 30001
    targetPort: 8080
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
$ curl http://192.168.0.192:30001
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
$kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE    SELECTOR
hello-node   NodePort    10.103.218.93   <none>        8080:30001/TCP   6m9s   service-name=hello-node
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          38m    <none>
```

### node port
nodeport는 전체 노드에 특정 포트 (30000번부터-)를 외부에 오픈한다. yml을 한번 보기 바란다. 

그러므로 다음 두 커맨드는 모두 동작한다.
```
curl http://192.168.0.192:30001
curl http://192.168.0.193:30001
```

클러스터의 모든 서버 아이피에 포트를 연다.

외부에 오픈하기는 가장 쉬운 방법 - 단점도 있다 포트가 30000번이상과 2개의 다른 서비스에 하나의 포트를 사용할수 없다.  전체 클러스터에 포트를 열기 때문 

### ClusterIP
clusterip가 기본값이다. 처음에는 이걸 왜 쓰냐고 생각햇는데 내부 서비스끼리 연결될때 이걸 사용한다. 

예를 들면 워드프레스와 mysql 두개의 서비스가 돌고 잇을때 mysql은 외부에 오픈할 필요없이 워드프레스 pod에만 연결이 되면 되므로 mysql 서비스를만들어서 clusetip로 하면된다. 

그럼 워드프레스에서 mysql에 어떻게 연결하나? - 나중에 해보다 //todo 

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

vi mt-config.yml

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
      - 192.168.0.80/28 
```
서비스에 줄 아이피를 정해뒀다 사용 가능한 아이피는 192.168.0.81 - 192.168.0.94 가 된다. 적용해보자.

```bash
kubectl create -f  mt-config.yml
# 상태를 확인해보자.
kubectl get pods -n metallb-system
kubectl logs -l component=speaker -n metallb-system
kubectl get svc -n metallb-system
```

이제 hello-node.yml을 수정해서 loadbalance로 변경해보자.

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
  type: LoadBalancer                # 여기만 수정
  loadBalancerIP: 192.168.0.81     # 여기만 수정
  ports:
  - port: 8080
    nodePort: 30001
    targetPort: 8080
  selector:
    service-name: hello-node
```
kubectl delete -f hello-node.yml
kubectl create -f hello-node.yml

```bash
$ kubectl  get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.106.215.59   <pending>     8080:30001/TCP   9s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          74m
```
type이 loadbalancer로 바귀었고 external-ip가 pending인것을 알수 있다. 

외부 아이피를 받을때까지 대기한다. ( 3분이 넘어가면 뭐가 잘못된것이므로 다시 해본다.)
```
$ kubectl  get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
hello-node   LoadBalancer   10.110.182.246   192.168.0.81   8080:30001/TCP   4s
kubernetes   ClusterIP      10.96.0.1        <none>         443/TCP          77m
```

외부 아이피를 잘 받았다. 이제 이 아이피를 호출해보면된다. 
```bash
curl http://192.168.0.81:8080 # 포트가 서비스 자체 포트로 바귀었다.
```

### metallb 삭제하고 싶으면 다음처럼 하면된다. 

```
kubectl delete -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
kubectl delete -f mt-config.yml
kubectl get configmap -n metallb-system
kubectl get svc 
```

### sample pod and service 삭제 
```bash
kubectl delete pods hello-node
kubectl delete svc hello-node
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

kubernetes를사용할때는 꼭 name space를 쓰기를 추천드립니다.
```
kubectl create namespace publish-api-dev
kubectl create namespace publish-api-live
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
namespace: publish-api-dev
apiVersion: v1
metadata:
  name: dev-mysql-pv-volume
  namespace: publish-api-dev
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
  namespace: publish-api-dev
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
vi dev-mysql-deployment.yml
```
```yml
---
apiVersion: v1
kind: Service
metadata:
  name: dev-mysql
  namespace: publish-api-dev
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.0.82
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: dev-mysql
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: dev-mysql
  namespace: publish-api-dev
spec:
  selector:
    matchLabels:
      app: dev-mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: dev-mysql
    spec:
      containers:
      - image: mysql:5.6
        name: dev-mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value:
            password
        ports:
        - containerPort: 3306
          name: dev-mysql
        volumeMounts:
        - name: dev-mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: dev-mysql-persistent-storage
        persistentVolumeClaim:
          claimName: dev-mysql-pv-claim
```
```
kubectl create -f dev-mysql-pv-pvc.yml 
kubectl get pv -n publish-api-dev
kubectl get pvc -n publish-api-dev
kubectl create -f dev-mysql-deployment.yml
kubectl get pods -n publish-api-dev
kubectl get services -n publish-api-dev
```

Node192 번에 /data/dev-mysql폴더가 없으면 만들어 줘야한다. 

여기에 데이터를 저장하게 해두었으나 실제로는 nfs등에 저장하면될듯 

개발디비여서 외부에서 접속할 필요가 있어서 loadbalancer 를 서비스 타입으로 사용하나 서비스 디비는 외부에 오픈될 필요가 없으면 cluseterip를 사용해도 될듯 싶다. 


## ingress-nginx 

하나의 아이피에 각각의 서비스로 보내주고 싶다.

인그레스에 연결될 서비스를 하나 만들어 보자. 위에서 사용한 hello-node.yml을 사용한다.

vi hello-node.yml
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
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: hello-node
```

kubectl create -f hello-node.yml

서비스 타입은 기본값인 clusterip를 사용햇다.

인그레스를 nodeport type의 서비스로 만들어보자.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```
잘됬는지 체크 
```bash
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx 
kubectl get svc -n ingress-nginx  #open된 포트 확인하자
```

설정을 추가하자.

vi ingress-config.yml
```yml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  # namespace: publish-api-live
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: publishapi.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hello-node
          # serviceName: auth-svc
          servicePort: 8080
          
  - host: auth.publishapi.com
    http:
      paths:
      - backend:
          serviceName: hello-node
          # serviceName: auth-svc
          servicePort: 8080
```
kubectl create -f ingress-config.yml


확인해보자. 

curl http://192.168.0.192:31531

동작한다.

## 로드발란스 타입의 ingress 서비스로 바꿔보자. 

위에서 사용한 metallb가 꼭 필요합니다. 

기존 서비스를 지우자.
```
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml

vi ingress-service.yml
```

```yml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: LoadBalancer               # 이부분만 수정됨
  loadBalancerIP: 192.168.0.83     # 이부분만 수정됨
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

적용하고 확인해보자.
```
kubectl create -f ingress-service.yml
kubectl get svc -n ingress-nginx 
```

로드발란스로 동작한다.

http://api.publishapi.com/healthz


## Spinnaker

스피네커는 젠킨스와 비슷한 역할을 수행한다. halyard가  kubernetes cluser에 접속해서 설치를 진행한다. 그러므로 kubernetes config파일을 복사해서 halyard노드에 추가해야한다.

* on master
```
cat ~/.kube/config
```
복사해둔다. 

쿠버네티스 클러스터가 현재 존재하고 이제 halyard를 설치할 서버를 준비하자. node194 에서 다음 작업을 진행한다.

### halyard (on 194)
```bash
mkdir ~/.hal
chmod 777 -R ~/.hal

mkdir ~/.kube
vi ~/.kube/config #master에서 내용을 가져와서 붙여넣기해야한다. 
chmod -R  777 ~/.kube
```

도커 이미지가 있어서 바로 실행하면 된다.
```
docker run -p 8084:8084 -p 9000:9000 \
    --name halyard \
    -v ~/.hal:/home/spinnaker/.hal \
    -v ~/.kube:/home/spinnaker/.kube \
    -d \
    gcr.io/spinnaker-marketplace/halyard:stable
```
도커 이미지가 상당히 크다.

컨테이너에 접속하자. 
```bash
docker exec -it halyard bash
```

다음은 탭 완성 기능 
```
source <(hal --print-bash-completion)
```

### kubernetes-v2 를 대상으로 배포해야하므로 master에 작업을 한다. 

* on master

```
kubectl create namespace spinnaker

vi spinnaker.yml
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

### Adding an account (on node194 halyard container 안에서)

First, make sure that the provider is enabled:
```
hal config provider kubernetes enable
```

Then add the account:
```
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --context $CONTEXT
```

You’ll also need to run
```
hal config features edit --artifacts true
```

Add Account
```
hal config deploy edit --type distributed --account-name my-k8s-v2-account
```

## minio
스피네커에서 사용하는 내용을 저장하기위해 외부 저장소를 쓰는데 여기서는 간단하게 minio를 사용하기로 한다. 

s3도 안쓰고 node194에 로컬에 그냥 저장하는걸로 하자.

### 미니오 서버를 실행 (on node194)

secret key와 access key를 볼수 있다. 
```bash
mkdir -p /data/minio
mkdir -p /data/minio-config
cd
wget https://dl.minio.io/server/minio/release/linux-amd64/minio
chmod +x minio
mv minio /usr/bin/
minio server --address ":9001" --config-dir /data/minio-config /data/minio

> Endpoint:  http://192.168.0.194:9001  http://172.17.0.1:9001  http://127.0.0.1:9001
> AccessKey: OS2PVUL53ZTSNMJOWOWR
> SecretKey: kLP1IdRqS+WqaLJ6WOXjrq80LptXy+j9SoeqXRLs
> 
> Browser Access:
>    http://192.168.0.194:9001  http://172.17.0.1:9001  http://127.0.0.1:9001
> 
```

서버가 실행되게 둔상태에서 halyard 컨테이너로 간다. 

```
MINIO_ACCESS_KEY=OS2PVUL53ZTSNMJOWOWR
MINIO_SECRET_KEY=kLP1IdRqS+WqaLJ6WOXjrq80LptXy+j9SoeqXRLs
ENDPOINT=http://192.168.0.194:9001

echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key 

hal config storage edit --type s3
```

s3로 하는 이유는 아마존과는 상관이 없고 minio가 s3와 compatible 하기 때문이다. 

그러나 versioning은 지원하지 않아서 다음 커맨드를 실행해야하는데 도커 컨테이너가 vi가 없다. 

그래서 node194로 터미널을 하나더 열어서 다음을 실행한다.

```
mkdir -p ~/.hal/default/profiles/
vi ~/.hal/default/profiles/front50-local.yml
spinnaker.s3.versioning: false
```

### Deploy Spinnaker and Connect to the UI 

컨테이너로 접속해서 
```bash
docker exec -it halyard bash
hal version list #사용할 버전을 고른다. 난 1.10.6

hal config version edit --version 1.10.6
```

Deploy Spinnaker
```
hal deploy apply
```

halyard가 쿠베에 접속해서 디플로이를 순서대로 한다. 

### Connect to the Spinnaker UI - 화면을 보자.
```
hal deploy connect
```


This command automatically forwards ports 9000 (Deck UI) and 8084 (Gate API service).

웹브라우저로 접속해보자. 

http://192.168.0.194:9000.



hal config security ui edit --override-base-url http://192.168.0.194:9000
hal config security api edit --override-base-url http://192.168.0.194:8084






























































## helm 

### installing helm 
```bash
cd 
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

helm help
```

### Role-based Access Control
TILLER AND ROLE-BASED ACCESS CONTROL

vi rbac-config.yaml
```yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
```
kubectl create -f rbac-config.yaml
```

### install Tiller
Tiller, the server portion of Helm, typically runs inside of your Kubernetes cluster.
```
helm init --service-account tiller
```

### Using Helm

* helm search 
* helm inspect stable/mariadb
* helm install stable/mariadb -n kubelive
* helm status happy-panda
* helm list
* helm list --deleted
* helm list --all
* helm delete happy-panda
* helm delete --purge
* helm reset 
* helm upgrade -f panda.yaml happy-panda stable/mariadb
* helm get values happy-panda
* helm rollback [RELEASE] [REVISION].
* helm rollback happy-panda 1
* helm repo list
* helm repo add dev https://example.com/dev-charts

### Customizing the Chart Before Installing
* helm inspect values
```bash
echo '{mariadbUser: user0, mariadbDatabase: user0db}' > mariadb-config.yaml
helm install -f mariadb-config.yaml stable/mariadb
```


























