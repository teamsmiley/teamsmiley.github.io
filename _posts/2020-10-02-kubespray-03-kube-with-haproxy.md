---
layout: post
title: "kubespray - 03 install kube with haproxy"
author: teamsmiley
date: 2020-10-02
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubespray }
---

# kubespray - 03 install kube - haproxy 버전

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>

2개의 서버로 haproxy를 설정하여 하나가 문제가 생겨도 서비스에는 지장이 없게 한다.

쿠버네티스가 haproxy를 보고 있게 한다. 맨 마지막 다이어그램 참고

## vm 셋업

vi Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "provision.sh"

  # master
  config.vm.define "minion1" do |minion1|
    minion1.vm.box = "centos/7"
    minion1.vm.hostname = "minion1"
    minion1.vm.network "private_network", ip: "192.168.33.21"
    minion1.vm.provider "virtualbox" do |v|
      v.memory =1501
      v.cpus = 1
	  end
  end

  # master
  config.vm.define "minion2" do |minion2|
    minion2.vm.box = "centos/7"
    minion2.vm.hostname = "minion2"
    minion2.vm.network "private_network", ip: "192.168.33.22"
    minion2.vm.provider "virtualbox" do |v|
      v.memory =1501
      v.cpus = 1
	  end
  end

  config.vm.define "minion3" do |minion3|
    minion3.vm.box = "centos/7"
    minion3.vm.hostname = "minion3"
    minion3.vm.network "private_network", ip: "192.168.33.23"
    minion3.vm.provider "virtualbox" do |v|
      v.memory =1501
      v.cpus = 1
	  end
  end

  config.vm.define "haproxy01" do |haproxy01|
    haproxy01.vm.box = "centos/7"
    haproxy01.vm.hostname = "haproxy01"
    haproxy01.vm.network "private_network", ip: "192.168.33.2"
    haproxy01.vm.provider "virtualbox" do |v|
      v.memory = 512
      v.cpus = 1
	  end
  end

  config.vm.define "haproxy02" do |haproxy02|
    haproxy02.vm.box = "centos/7"
    haproxy02.vm.hostname = "haproxy02"
    haproxy02.vm.network "private_network", ip: "192.168.33.3"
    haproxy02.vm.provider "virtualbox" do |v|
      v.memory = 512
      v.cpus = 1
	  end
  end

end

vagrant up

# 자동 로그인
ssh-copy-id 192.168.33.2
ssh-copy-id 192.168.33.3
ssh-copy-id 192.168.33.23
ssh-copy-id 192.168.33.22
ssh-copy-id 192.168.33.23
```

haproxy vm은 192.168.33.2 192.168.33.3 번 아이피를 사용

이 두개의 아이피를 대표하는 vip는 192.168.33.10으로 사용 예정

## install haproxy && keepalived

아래 링크에서 보고 설치하면 된다.

<https://teamsmiley.github.io/2020/10/02/haproxy-keepalived/>

## seliux off

```bash
vi /etc/sysconfig/selinux

> SELINUX=disabled

reboot
```

## setup haproxy01 haproxy02

두개다 설정 한다.

vi /etc/haproxy/haproxy.cfg

```ruby
listen kubernetes-apiserver-https
  bind :8383

  mode tcp
  option log-health-checks
  timeout client 3h
  timeout server 3h

  # server master1 <IP1>:6443 check check-ssl verify none inter 10000
  # server master2 <IP2>:6443 check check-ssl verify none inter 10000
  # server master3 <IP2>:6443 check check-ssl verify none inter 10000

  server master1 192.168.33.21:6443 check check-ssl verify none inter 10000
  server master2 192.168.33.22:6443 check check-ssl verify none inter 10000
  server master3 192.168.33.23:6443 check check-ssl verify none inter 10000
  balance roundrobin
```

```bash
systemctl restart haproxy
systemctl status haproxy
```

### error

```
Oct 01 19:30:03 haproxy02 haproxy-systemd-wrapper[600]: [ALERT] 274/193002 (603) : Starting proxy kubernetes-apiserver-https: cannot bind socket [192.168.33.10:8383]
```

bind 192.168.33.10:8383 => bind :8383

이렇게 고치니 에러 사라짐

## kubespray 설정

전체 노드에서 인터넷이 되는지 확인하자.

`ping google.com`

External LB 설정 부분을 수정하자.

vi inventory/mycluster/group_vars/all/all.yml

```yml
## External LB example config
apiserver_loadbalancer_domain_name: "haproxy.xgridcolo.com"
loadbalancer_apiserver:

## Internal loadbalancers for apiservers
loadbalancer_apiserver_localhost: false # 여기 주석해제후 수정
# valid options are "nginx" or "haproxy"
# loadbalancer_apiserver_type: nginx  # valid values "nginx" or "haproxy"
```

kubespray가 노드에 /etc/hosts파일에 이부분을 자동으로 다 넣어준다.

laptop은 수동으로 넣어준다.

vi /etc/hosts
192.168.33.10 haproxy.xgridcolo.com

## ansible

vi inventory/mycluster/hosts.yml

정리해서 사용하는 vm만 남겨두면 된다.

이제 실행해보자.

```bash
ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml

mkdir .kube
scp 192.168.33.21:/etc/kubernetes/admin.conf ~/.kube/config

kubectl version
kubectl cluster-info

> Kubernetes master is running at https://haproxy.xgridcolo.com:8383

kubectl config view
```

여기에서 server가 정보가 어떻게 되잇는지 확인해보자.

![]({{ site_baseurl }}/assets/2020-10-01-14-07-45.png)

잘되있다.

## 다이어그램

![]({{ site_baseurl }}/assets/2020-10-01-10-15-15.png)

이제 쿠버네티스 쉽게 설치하자.
