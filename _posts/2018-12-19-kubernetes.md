---
layout: post
title: 'kubernetes 초급 - 설치하고 사용하기' 
author: teamsmiley
date: 2018-12-19
tags: [devops]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# 기본 설치 프로그램을 설치합니다.

## homebrew install
https://brew.sh/index_ko

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## home brew cask 

Homebrew-Cask은 Google 크롬 또는 Atom과 같은 GUI 응용 프로그램을 설치하는 Homebrew의 확장 프로그램입니다. 

독립적으로 시작되었지만 유지 보수 담당자는 Homebrew의 핵심 팀과 긴밀하게 협력합니다.

```bash
brew install brew-cask
```

## virtualbox install
가상머신을 설치하기 위한 프로그램입니다.
```bash
brew cask install virtualbox
```

## vagrant install
virtual box를 관리하기 위해서 사용합니다. 조금 쉽게 virtualbox를 사용가능합니다.

```bash
brew cask install vagrant
```

## kubernetes 설치 

1개의 마스터 서버 (centos 7)
2개의 노드 서버 (centos 7)
1개의 미니오 스토리지 서버(centos 7)
1개의 도커 registry (centos 7)

아이피는 다음과 같습니다. 

| | |
|---|---|
192.168.0.191 | master  |
192.168.0.192 | node192 |
192.168.0.193 | node193 |
192.168.0.194 | node194 | 
192.168.0.195 | registry |
| | |

## 맥북 wifi를 192.168.0.1로 지정
```
sudo ifconfig en0 alias 192.168.0.1/24 up
```

## 설치 - Master,Node192,Node193,Node194 

## 가상머신 만들기 

마스터 서버를 설치해봅니다.

```
mkdir -p ~/Desktop/kube/master
cd ~/Desktop/kube/master
vagrant init centos/7 --minimal
vagrant up
```

virtual box를 실행해서 vm 이 만들어진것을 확인한다.

이제 vm에 이름과 아이피를 지정하자. 

Vagrantfile 파일을 수정하면 됩니다. 

```ini
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  # 추가 부분
  config.vm.network "public_network", ip: "192.168.0.191", bridge: "en0: Wi-Fi (AirPort)"
  config.vm.hostname "master"
  # cpu 2개 momory 4G kubernetes 요구사항
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", "4000"]
    v.customize ["modifyvm", :id, "--cpus", "2"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

end
```

vagrant 로 다시 vm을 시작해봅니다.

```bash
vagrant halt # vm을 정지시킵니다.
vagrant destroy -y # vm을 삭제합니다.
vagrant up # 새로운 설정으로 vm 을 부팅시킵니다.
vagrant plugin install vagrant-vbguest # 추가 플러그인 설치합니다.
vagrant ssh # 가상머신으로 접속합니다.
sudo yum update && sudo yum -y install kernel-headers kernel-devel #이거 안하면 mount에러남
sudo yum install net-tools -y # ifconfig를 설치합니다. 
hostname # hostname 을 체크합니다.
```

kubernetes를 설치하기전 해야할 일이 있습니다. 

<https://kubernetes.io/docs/setup/independent/install-kubeadm/#before-you-begin>

문서를 참고하시면되는데요 진행해보겠습니다. 

## before you begin kubernetes

```bash
# hostname 설정
sudo hostnamectl set-hostname 'master'
# selinux off
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# firewall off
sudo systemctl disable firewalld

# docker install
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker vagrant
sudo systemctl start docker
sudo systemctl enable docker

sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo modprobe br_netfilter
```
vagrant 를 재시작하면 /proc/sys/net/bridge/bridge-nf-call-iptables 값이 항상 0으로 바뀐다. 매번 1로 설정해줘야한다.
```
sudo bash 
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
exit
```

<!-- 이걸 좀 쉽게 하는법이 있다 vagrant가 부팅되면서 항상 실행되는 파일을 만들면 된다. 

vi startup.sh

```bash
#!/bin/bash
sudo bash
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

vi Vagrantfile

```ini
# 추가
config.vm.provision "shell", path: "startup.sh"
```

확인해보자. 

```
vagrant reload
vagrant ssh 
cat /proc/sys/net/bridge/bridge-nf-call-iptables
``` -->





* host 파일 설정
```
sudo vi /etc/hosts
```
```
192.168.0.101 master
192.168.0.192 node192
192.168.0.193 node193
192.168.0.194 node194
192.168.0.195 registry
```

* swap off
```bash
sudo swapoff -a # 임시로 swap off 재부팅시 살아남.
sudo vi /etc/fstab  # swap을 지우기 
```

## kubernetes package install

```bash
sudo bash
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

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable kubelet 
sudo systemctl start kubelet
exit
exit
```
이렇게 하면 kubernetes설치까지 마무리 된것이다. 이제 이 vm을 이미지로 만들어서 node192 node192을 만들어보자.

## vm이미지 만들어서 나머지 노드들 만들기

위처럼 똑같이 2개를 더 만들어서 설치하면되긴 하는데 귀찮아서 이미지로 만들어서 바로 로딩해서 사용할수 있게 합니다.

```bash
vagrant package # package.box가 생성됩니다. 
vagrant box add kube-default package.box # package.box를 이름으로 등록합니다.
rm -f package.box # 다른 컴퓨터에서 사용하려면 옮겨 둬야 한다.
```

## node192 setup 
192노드를 셋업합니다.

```bash
mkdir -p ~/Desktop/kube/node192
cd ~/Desktop/kube/node192
vagrant init kube-default --minimal
vagrant plugin install vagrant-vbguest
vagrant up
vagrant ssh 
ifconfig # ip 확인
kubectl --help
```
kubectl이 설치되있는것을 확인할수 있습니다. 

이제 아이피를 지정해봅시다.
```
vi Vagrantfile
```
```ini
Vagrant.configure("2") do |config|
  config.vm.box = "kube-default"
  # 추가된 부분
  config.vm.hostname = "node192"
  config.vm.network "public_network", ip: "192.168.0.192", bridge: "en0: Wi-Fi (AirPort)"
end
```

* 혹시 다음 에러가 나시면 업데이트가 필요합니다.

```
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

The error output from the command was:

/sbin/mount.vboxsf: mounting failed with the error: No such device
```
* 해결방법
```
vagrant ssh 
sudo yum update && sudo yum -y install kernel-headers kernel-devel
exit
vagrant reload 
```

## node193을 설치

```bash
mkdir -p ~/Desktop/kube/node193
cd ~/Desktop/kube/node193
vagrant init kube-default --minimal

vi Vagrantfile

> Vagrant.configure("2") do |config|
>   config.vm.box = "kube-default"
>   # 추가된 부분
>   config.vm.hostname = "node193"
>   config.vm.network "public_network", ip: "192.168.0.193", bridge: "en0: Wi-Fi (AirPort)"
> end

vagrant up # 혹시 에러나면 플러그인 설치
vagrant plugin install vagrant-vbguest
vagrant up
vagrant ssh 
ifconfig # ip 확인
kubectl --help
```
kubectl이 설치되있는것을 확인할수 있습니다. 

## registry도 설치
```bash
mkdir -p ~/Desktop/kube/registry/data/docker
mkdir -p ~/Desktop/kube/registry/data/registry
cd ~/Desktop/kube/registry
vagrant init kube-default --minimal

vi Vagrantfile

> Vagrant.configure("2") do |config|
>   config.vm.box = "kube-default"
>   # 추가된 부분
>   config.vm.hostname = "registry"
>   config.vm.network "public_network", ip: "192.168.0.195", bridge: "en0: Wi-Fi (AirPort)"
>   config.vm.network "forwarded_port", host: 5000, guest: 5000
>   config.vm.synced_folder "./data/", "/data/"
> end

```

* vm의 포트 5000번과 로컬에 포트 5000을 매핑
* ./data 폴더를 vm에 /data 로 매핑을 해준다.

```bash
vagrant up # 혹시 에러나면 플러그인 설치
vagrant plugin install vagrant-vbguest
vagrant up

vagrant ssh 
ifconfig # ip 확인
exit
```
이제 repository를 도커를 이용하여 실행해보자. 

```bash
vi ~/Desktop/kube/registry/data/docker/docker-compose.yml
```
```yml
---
version: "3.3"

services: 
  registry:
    container_name: registry
    restart: always
    image: registry:2.6.2
    privileged: true
    ports:
      - 5000:5000
    environment:
      TZ: "America/Los_Angeles"
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data/registry
    volumes:
      - /data/registry:/data/registry/docker/registry
```
데이터는 이 /data/registry 경로에 저장을 한다.

이제 도커를 실행해보자.

```bash
vagrant reload # 설정이 바뀌였으므로 재시작한다.
cd ~/Desktop/kube/registry/
vagrant ssh

cd /data/docker && docker-compose up -d
docker ps 
```

docker private registry를 생성하자. 도커 이미지를 만들면 저장해두는 저장소다. 도커를 빌드해서 사용할 계획이므로 이것이 꼭 먼저 있어야한다.

이렇게 하면 도커 이미지들을 저장할수 있다.

## minio server
spinnaker 에서 필요한 데이터를 저장하는 저장소 역할을 합니다. s3등을 사용하여도 되나 여기서는 vm으로 저장합니다. 

```bash
mkdir ~/Desktop/kube/minio/data/minio
mkdir ~/Desktop/kube/minio/data/minio-config

cd ~/Desktop/kube/minio
vagrant init kube-default --minimal
vagrant up 

vi Vagrantfile

> Vagrant.configure("2") do |config|
>   config.vm.box = "kube-default"
>   # 추가된 부분
>  config.vm.hostname = "minio"
>  config.vm.network "public_network", ip: "192.168.0.194", bridge: "en0: Wi-Fi (AirPort)"
>  config.vm.synced_folder "./data/", "/data/"
> end

vagrant up 
```

## 전체 설치 버전

<https://github.com/heptiolabs/wardroom/blob/master/swizzle/Vagrantfile>

```bash
vi Vagrant file 
vagrant plugin install vagrant-libvirt
vagrant up 
```



## init kubenetes - master

이제 쿠버네티스를 셋업해보자. 

```bash
cat /proc/sys/net/bridge/bridge-nf-call-iptables
sudo bash 
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
cat /proc/sys/net/bridge/bridge-nf-call-iptables

cat /proc/sys/net/ipv4/ip_forward
echo '1' > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward

route del default eth0 # 기본 라우터를 eth1 192로 한다.
route add default gw 192.168.0.1 netmask 0.0.0.0 dev eth1 # default gw 추가 

kubeadm init
```
결과값을 잘 복사해두자. 나중에 이 값을 이용해서 노드를 마스터에 연결해준다.

```
You can now join any number of machines by running the following on each node
as root:

kubeadm join 192.168.0.191:6443 --token a24iv1.35ela7tz6sfc1h1r --discovery-token-ca-cert-hash sha256:432645b936c5103fc97f79ecead7340f2a766627750c13bfad72e09d246a3567
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
vagrant up
vagrant ssh

sudo bash 

cat /proc/sys/net/bridge/bridge-nf-call-iptables
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
cat /proc/sys/net/bridge/bridge-nf-call-iptables

cat /proc/sys/net/ipv4/ip_forward
echo '1' > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward

sudo kubeadm join 192.168.0.191:6443 --token lr98l5.962xm2vi5pznrdhz --discovery-token-ca-cert-hash sha256:d1ffcec6e71cc2be3105adf450d80f179462db35abe47155820bab852ce1d6f5
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
  name: ingress-xxx  
  namespace: ingress-nginx # 이부분 아래 설명 참고 - A
  labels:
    app.kubernetes.io/name: ingress-nginx # 이부분 아래 설명 참고 - A
    app.kubernetes.io/part-of: ingress-nginx # 이부분 아래 설명 참고 - A
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
    app.kubernetes.io/name: ingress-nginx # 이부분 아래 설명 참고 - A
    app.kubernetes.io/part-of: ingress-nginx # 이부분 아래 설명 참고 - A
```

적용하고 확인해보자.
```
kubectl create -f ingress-service.yml
kubectl get svc -n ingress-nginx 
```

로드발란스로 동작한다.

* 중요 

헷갈린 부분이 있어서 정리해둔다. 

일단 서비스로 올리는 컨테이너는 # 이부분 아래 설명 참고 - A 이부분을 바꾸지 말기 바란다. 

바꿀수도 있지만 일이 복잡해진다.  이름만 바꾸고 쓰자. 

그러나 실제 연결 되는 서비스는 namespace가 위와 달라도  서비스명으로 그냥 연결할수 있다. 아래 스피네커 붙이는 부분 참고 

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



### docker private registry enable 

private registry 에 꼭 ssl이 필요하다 그것도 selfsign이 아닌 제대로 된것. 아이디 비번도 꼭 필요하다.

그래서 let's encrypt로 설치했다. 

참고 <https://teamsmiley.github.io/2018/12/22/docker-private-registry/>

```bash

docker exec -it halyard bash

CONTEXT=$(kubectl config current-context)

hal config provider docker-registry enable

ADDRESS=registry.ur-domain.com:5000 
REPOSITORIES="auth-server"
USERNAME=ur-username

hal config provider docker-registry account add my-registry \
    --repositories $REPOSITORIES \
    --address $ADDRESS \
    --username $USERNAME \
    --password 
# password는 프롬프트로 물어본다.

hal config provider kubernetes account add my-k8s-v2-account 
```

### Adding an account (on node194 halyard container 안에서)

First, make sure that the provider is enabled:
```
hal config provider kubernetes enable
```

Then add the account:
```bash
CONTEXT=$(kubectl config current-context)

hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --docker-registries my-registry # registry에서 이미지를 받을수 있게
    --context $CONTEXT
```

You’ll also need to run
```bash
hal config features edit --artifacts true
# Add Account
hal config deploy edit --type distributed --account-name my-k8s-v2-account
```

## minio

스피네커에서 사용하는 내용을 저장하기위해 외부 저장소를 쓰는데 여기서는 간단하게 minio를 사용하기로 한다. 

s3도 안쓰고 node194에 로컬에 그냥 저장하는 걸로 하자.

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

3번이 제일 쉽니다.

2 3 번은 설정 변경후 아래 커맨드를 실행해줘서 컨테이너들에 알맞는 설정이 들어가야한다. 

```bash
docker exec -it halyard bash

hal config security ui edit --override-base-url http://spinnaker-ui
hal config security api edit --override-base-url http://spinnaker-gate
hal deploy apply
```

그럼 인그레스를 이용해서 해보자. 

### ingress 이용

vi ingress-spin.yml

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
```

```
kubectl create -f ingress-spin.yml
```

hosts파일에 설정을 하자.

vi /etc/hosts
```
192.168.0.84 spinnaker-ui spinnaker-gate
```

http://spinnaker-ui

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

add stage >> deploy >> Manifest Source >> text 

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

끝

