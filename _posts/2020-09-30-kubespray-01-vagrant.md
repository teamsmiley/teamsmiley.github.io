---
layout: post
title: "kubespray - 01 vagrant & virtual box"
author: teamsmiley
date: 2020-09-30
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubespray }
---

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>

# kubespray - 01 vagrant & virtual box

## 설치

vagrant가 잇어야 로컬에서 테스트가 잘 될듯

virtual box (꼭 version 6.0을 사용) && vagrant downinstall

https://www.virtualbox.org/wiki/Downloads

https://www.vagrantup.com/downloads.html

둘다 설치한다.

virtualbox 6.1 은 macos에서 잘 동작하지 않음...

## vagrantfile 생성

```bash
mkdir kubespray
vagrant init centos/7 --minimal
cd kubespray
vi Vagrantfile
```

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
end
```

```
vagrant up
```

## 생성된 vm에 접속

```
vagrant ssh
```

vagrant 유저와 비번 vagrant로 접속되는듯.

## vm 여러대를 만들어보자.

ip를 지정해서 만들어보자.

```ruby
Vagrant.configure("2") do |config|

    config.vm.define "minion1" do |minion1|
    minion1.vm.box = "centos/7"
    minion1.vm.hostname = "minion1"
    minion1.vm.network "private_network", ip: "192.168.33.21"
  end

  config.vm.define "minion2" do |minion2|
    minion2.vm.box = "centos/7"
    minion2.vm.hostname = "minion2"
    minion2.vm.network "private_network", ip: "192.168.33.22"
  end

  config.vm.define "minion3" do |minion3|
    minion3.vm.box = "centos/7"
    minion3.vm.hostname = "minion3"
    minion3.vm.network "private_network", ip: "192.168.33.23"
  end

  config.vm.define "minion4" do |minion4|
    minion4.vm.box = "centos/7"
    minion4.vm.hostname = "minion4"
    minion4.vm.network "private_network", ip: "192.168.33.24"
  end

end
```

ping 192.168.33.10

핑 결과가 온다. 아이피는 설정 완료

## root로 접속

나는 root로 접속하는게 좋다. 그래서 설정해봣다.

vi Vagrantfile

```ruby

  config.vm.define "minion1" do |minion1|
    minion1.vm.box = "centos/7"
    minion1.vm.hostname = "minion1"
    minion1.vm.network "private_network", ip: "192.168.33.21"
    minion1.vm.provider "virtualbox" do |v|
      # master로 사용됨 kubespray가 마스터노드는 메모리가 1500보다 커야함.
      v.memory =1501
      v.cpus = 1
	  end
  end

  config.vm.define "minion2" do |minion2|
    minion2.vm.box = "centos/7"
    minion2.vm.hostname = "minion2"
    minion2.vm.network "private_network", ip: "192.168.33.22"
    minion2.vm.provider "virtualbox" do |v|
    # master로 사용됨 kubespray가 마스터노드는 메모리가 1500보다 커야함.
      v.memory =1501
      v.cpus = 1
	  end
  end

  config.vm.define "minion3" do |minion3|
    minion3.vm.box = "centos/7"
    minion3.vm.hostname = "minion3"
    minion3.vm.network "private_network", ip: "192.168.33.23"
    minion3.vm.provider "virtualbox" do |v|
    # kubespray가 워커 노드는 메모리가 1024보다 커야함.
      v.memory =1025
      v.cpus = 1
	  end
  end

  config.vm.define "minion4" do |minion4|
    minion4.vm.box = "centos/7"
    minion4.vm.hostname = "minion4"
    minion4.vm.network "private_network", ip: "192.168.33.24"
    minion4.vm.provider "virtualbox" do |v|
      # kubespray가 워커 노드는 메모리가 1024보다 커야함.
      v.memory =1025
      v.cpus = 1
	  end
  end

  # 나중에 추가할 노드
  # config.vm.define "minion5" do |minion5|
  #   minion5.vm.box = "centos/7"
  #   minion5.vm.hostname = "minion5"
  #   minion5.vm.network "private_network", ip: "192.168.33.25"
  #   minion5.vm.provider "virtualbox" do |v|
  #     # kubespray가 워커 노드는 메모리가 1024보다 커야함.
  #     v.memory =1025
  #     v.cpus = 1
	#   end
  # end
```

vi provision.sh

```bash
# root password set
echo -e "YourPassword\nYourPassword" | passwd
# root login allow
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config;
sed  -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;

echo "PS1='[\e[1;32m\t\e[0m][\e[1;33m\u\e[0m@\e[1;36m\h\e[0m \w] \n\$ \[\033[00m\]'" >> /etc/bashrc

reboot;
```

provision.sh는 항상 루트로 실행.

```bash
vagrant destroy -f
vagrant up
```

ssh 192.168.33.10

비번 넣고 로그인하면 된다

## 자동 로그인

```bash
ssh-copy-id 192.168.33.21
ssh-copy-id 192.168.33.22
ssh-copy-id 192.168.33.23
ssh-copy-id 192.168.33.24
ssh 192.168.33.21
ssh 192.168.33.22
ssh 192.168.33.23
ssh 192.168.33.24
```

비번없이 로그인되면 성공

## 생성된 이미지를 저장해서 사용

vm에 이것저것 세팅을 다 해두고 그상태로 이미지로 만들고 싶다.

ssh키 등록을 다해두고 그걸 저장해서 이미지로 만들고 나중에 이 이미지를 사용하면 다시 등록하지 않아도 된다. 기타 기본 패키지도 다 설치해두고 싶다.

Vagrantfile이 있는 폴더에서 다음 명령어를 실행한다.

```
vagrant status
```

```
Current machine states:

minion1                   running (virtualbox)
minion2                   running (virtualbox)
minion3                   running (virtualbox)
haproxy01                 running (virtualbox)
haproxy02                 running (virtualbox)
```

원하는 vm을 선택후

```
vagrant package haproxy01 -- output haproxy01.box
```

이제 이 박스를 언제든 불러쓸수잇게 vagrant 에 추가하자.

```
vagrant box add haproxy01 haproxy01.box
vagrant box list
```

box박스는 잘 저장해두면 나중에 다른 컴에서도 사용이 가능

이제 Vagrantfile을 변경하자.

```ruby
# config.vm.provision "shell", path: "provision.sh"
# 이건 벌써된상태로 이미지를 만들어서 더이상 필요없어짐

config.vm.define "haproxy01" do |haproxy01|
    haproxy01.vm.box = "haproxy01" #centos/7에서 변경
    haproxy01.vm.hostname = "haproxy01"
    haproxy01.vm.network "private_network", ip: "192.168.0.108"
    haproxy01.vm.provider "virtualbox" do |v|
      v.memory = 512
      v.cpus = 1
	  end
  end
```

다시 vm을 올리자.

```
vagrant up
```

## 알고잇는 문제

- keepalived가 vagrant up할때 바로 안올라와서 매번 로그인해서 `systemctl restart keepalived`해주고 있다.

- vboxsf

```
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 vagrant /vagrant

The error output from the command was:
```

다음 설치하자.

```
vagrant plugin install vagrant-vbguest
```
