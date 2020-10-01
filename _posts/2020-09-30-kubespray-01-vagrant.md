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
