---
layout: post
title: "kubespray - 02 install kube - local internal loadbalancer"
author: teamsmiley
date: 2020-10-01
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubespray }
---

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>
1. <https://teamsmiley.github.io/2020/10/05/kubernetes-cert-manager/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-metallb-ingress-nginx/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-helm/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-prometheus-grafana/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-log/>

# kubespray - 02 install kube - local internal loadbalancer

## install python 3

python 3를 꼭 사용해야한다.

맥에 기본적으로 2.7이 설치되잇어서 상당히 헷갈릴수 있다.

https://www.python.org/downloads/mac-osx/

위 사이트에 가서 설치하자.

이제 pip3설치

```
cd
curl -O https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```

## kubespray git clone

https://github.com/kubernetes-sigs/kubespray

```bash
#git clone https://github.com/kubernetes-sigs/kubespray.git
git clone --depth 1 --branch v2.14.1 https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

python -V && pip -V
> Python 3.8.6
> pip 20.2.3 from /Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/pip (python 3.8)

# 3.0이상 사용
sudo pip install -r requirements.txt

# Copy ``inventory/sample`` as ``inventory/mycluster``
cp -rfp inventory/sample inventory/mycluster
## 또는 inventory를 따른 깃으로 관리하면
# cd inventory
# git clone ssh://git@gitlab.xgridcolo.com:30022/ragon/kubespray.git .

# 인벤토리 파일이 중요하다.
# Update Ansible inventory file with inventory builder
declare -a IPS=(192.168.33.21 192.168.33.22 192.168.33.23 192.168.33.24)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# 이 두개파일을 잘 보면 여러가지 설정을 변경할수 있다.
cat inventory/mycluster/group_vars/all/all.yml
cat inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
## 자동 로그인이 꼭 되있어야한다.
```

설치시 에러
메모리 가 작다는 에러

```
/extra_playbooks/roles/kubernetes/preinstall/defaults/main.yml
```

여기에 설정이 있다.

readme에도 있엇네

Hardware:
These limits are safe guarded by Kubespray. Actual requirements for your workload can differ. For a sizing guide go to the [Building Large Clusters](https://kubernetes.io/docs/setup/cluster-large/#size-of-master-and-master-components) guide.

- Master
  - Memory: 1500 MB
- Node
  - Memory: 1024 MB

virtualbox에서 메모리를 추가해주자. 1501 1025

다시 실행하면 잘 된다.

이제 다 된거같은데 확인해보자.

```
mkdir ~/.kube
scp root@minion1:/etc/kubernetes/admin.conf ~/.kube/config
k get nodes
kubectl version --short
kubectl cluster-info
kubectl -n kubesystem get pod

NAME                                          READY   STATUS    RESTARTS   AGE
calico-kube-controllers-c4bb85554-ccl7f       1/1     Running   0          21h
calico-node-6czfp                             1/1     Running   1          21h
calico-node-7mhz8                             1/1     Running   1          21h
calico-node-rdrtx                             1/1     Running   1          21h
calico-node-tp2tf                             1/1     Running   1          21h
coredns-8677555d68-7gv67                      1/1     Running   0          21h
coredns-8677555d68-hg45f                      1/1     Running   0          21h
dns-autoscaler-5b7b5c9b6f-ksnxq               1/1     Running   0          21h
kube-apiserver-node1                          1/1     Running   0          21h
kube-apiserver-node2                          1/1     Running   0          21h
kube-controller-manager-node1                 1/1     Running   0          21h
kube-controller-manager-node2                 1/1     Running   0          21h
kube-proxy-6ltgq                              1/1     Running   0          21h
kube-proxy-kpq52                              1/1     Running   0          21h
kube-proxy-n4jdt                              1/1     Running   0          21h
kube-proxy-np7d9                              1/1     Running   0          21h
kube-scheduler-node1                          1/1     Running   0          21h
kube-scheduler-node2                          1/1     Running   0          21h
kubernetes-dashboard-dfb67d98c-w47d9          1/1     Running   0          21h
kubernetes-metrics-scraper-54df648466-crwpd   1/1     Running   0          21h
nginx-proxy-node3                             1/1     Running   0          21h
nginx-proxy-node4                             1/1     Running   0          21h
nodelocaldns-4rrfw                            1/1     Running   0          21h
nodelocaldns-fz62v                            1/1     Running   0          21h
nodelocaldns-kbhll                            0/1     Pending   0          21h
nodelocaldns-z9r68                            1/1     Running   0          21h
```

여기서 확인해보면 nginx-proxy 가 있다. 이것이 바로 local internal loadbalance 이다. 마스터 하나가 죽으면 다른쪽으로 보내준다.

## add node

### vm한대 추가

vagrant에 1대 추가

```ruby
  config.vm.define "minion5" do |minion5|
    minion5.vm.box = "centos/7"
    minion5.vm.hostname = "minion5"
    minion5.vm.network "private_network", ip: "192.168.33.25"
  end
```

vagrant up

새로운 vm이 만들어진다.

```bash
ssh-copy-id 192.168.33.25 # 새 아이피로
```

### kubespray 설정

vi inventory/mycluster/hosts.yaml

```yml
...
    node5:
      ansible_host: 192.168.33.25
      ip: 192.168.33.25
      access_ip: 192.168.33.25
...
    kube-node:
      hosts:
        ...
        node4:
        node5: # 추가
```

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root scale.yml

PLAY RECAP *********************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node1                      : ok=470  changed=16   unreachable=0    failed=0    skipped=784  rescued=0    ignored=0
node2                      : ok=425  changed=14   unreachable=0    failed=0    skipped=556  rescued=0    ignored=0
node3                      : ok=367  changed=15   unreachable=0    failed=0    skipped=502  rescued=0    ignored=0
node4                      : ok=323  changed=12   unreachable=0    failed=0    skipped=441  rescued=0    ignored=0
node5                      : ok=350  changed=78   unreachable=0    failed=0    skipped=430  rescued=0    ignored=0

kubectl get nodes

node1   Ready    master   22h   v1.19.2
node2   Ready    master   22h   v1.19.2
node3   Ready    <none>   22h   v1.19.2
node4   Ready    <none>   22h   v1.19.2
node5   Ready    <none>   99s   v1.19.2
```

## remove node

### remove from cluster

```bash
watch kubectl get nodes # 다른 터미널에서 열어두고 보자.그럼 잘 보인다.

ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root remove-node.yml --extra-vars "node=node5"
#ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root remove-node.yml --extra-vars "node=node5,node6,node7,node8,node9"
```

### remove from hosts.yml

```yml
...
    # node5:
    #   ansible_host: 192.168.33.25
    #   ip: 192.168.33.25
    #   access_ip: 192.168.33.25
...
    kube-node:
      hosts:
        ...
        node4:
        # node5:
```

### delete vm

```bash
vagrant destroy minion5
```

## change version

```bash
kubectl get nodes

node1   Ready    master   22h   v1.19.2
node2   Ready    master   22h   v1.19.2
node3   Ready    <none>   22h   v1.19.2
node4   Ready    <none>   22h   v1.19.2

vi /inventory/mycluster/group_vars/k8s-cluster

kube_version: v1.19.2 #원하는 버전으로 수정한다.

ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root upgrade-cluster.yml
```

## reset cluster

kubespray가 설치되기 전으로 돌려준다.

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml
kubectl get nodes # 클러스터가 없으므로 에러
```

## 구성도

![]({{ site_baseurl }}/assets/2020-10-01-10-09-14.png)

노드들은 proxy가 있어서 하나의 마스터가 죽으면 자동으로 다음으로 넘어가지만 laptop은 마스터두개중 하나를 정해서 접속해야한다.
