---
layout: post
title: 'KeepAlived' 
author: teamsmiley
date: 2018-12-20
tags: [devops]
image: /files/covers/blog.jpg
category: {linux}
---

# keepalived 

서버 두대중 하나가 죽으면 다른 하나가 받아서 서비스를 한다.

## 기본 준비

centos 7으로 vm두개 설치했습니다. 

```bash
vagrant up
vagrant plugin install vagrant-vbguest
vagrant reload
vagrant ssh
```
두개의 노드 모두다 

```
sudo bash
yum install -y keepalived
systemctl enable keepalived
```

두개 노드 모두
```
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.old
vi /etc/keepalived/keepalived.conf
```
두 서버 같은 설정으로 넣는다.
```bash
vrrp_instance VI_1 {
  state MASTER
  interface eth1
  virtual_router_id 51
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {
    192.168.1.200/24 dev eth1 label eth1:0 # 이부분 중요
  }
}
```

서비스 재시작 후 로그 체크 
```bash
systemctl restart keepalived && tail -f /var/log/messages
```

테스트 
```
ping 192.168.1.200
```

마스터 노드를 재부팅해본다. 그래도 여전히 ping이 동작하면 성공

완료 


## haproxy를 사용중

haproxy를 사용중인데 서버가 재부팅이나 고장이 아니라 haproxy 서비스가 멈추는경우는 위에 경우로 확인이 안된다. 

설정을 추가해줘야한다.







