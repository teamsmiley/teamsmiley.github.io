---
layout: post
title: "haproxy keepalived"
author: teamsmiley
date: 2020-10-02
tags: [cicd]
image: /files/covers/blog.jpg
category: { haproxy }
---

# Haproxy and KeepAlived

## 구성

2개의 서버를 준비한다.

[새노드 설치 ](../xgrid/_New-Node-Install.md)

## ip 정보

| node name | ip           | memo                    |
| --------- | ------------ | ----------------------- |
| haproxy01 | 192.168.33.2 | keepalive 192.168.33.10 |
| haproxy02 | 192.168.33.3 | keepalive 192.168.33.10 |

## keepalived

### install

2개의 서버에 모두 같게 설치한다.

```bash
yum install -y keepalived
```

### config

```
vi /etc/keepalived/keepalived.conf
```

- master

```ruby
global_defs {
  notification_email {
    brian@xgridcolo.com
  }
  notification_email_from brian@xgridcolo.com
  smtp_server 127.0.0.1
  smtp_connect_timeout 30
  router_id LVS_DEVEL
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2 # every 2 seconds
  weight 2 # add 2 points if OK
}

vrrp_instance VI_1 {
  state MASTER
  interface eth1 # 본인 nic
  virtual_router_id 51
  priority 101 #Master
  virtual_ipaddress {
      192.168.33.10
  }
  track_script {
    chk_haproxy
  }
}
```

- slave

```ruby
global_defs {
  notification_email {
    brian@xgridcolo.com
  }
  notification_email_from brian@xgridcolo.com
  smtp_server 127.0.0.1
  smtp_connect_timeout 30
  router_id LVS_DEVEL
}

vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2 # every 2 seconds
  weight 2 # add 2 points if OK
}

vrrp_instance VI_1 {
  state BACKUP
  interface eth1
  virtual_router_id 51
  priority 100 # slave
  virtual_ipaddress {
      192.168.33.10
  }
  track_script {
    chk_haproxy
  }
}
```

```bash
systemctl enable keepalived
systemctl restart keepalived
```

확인

```bash
systemctl status keepalived
ping 192.168.33.10
```

33.2번 서버를 재부팅을 해도 여전히 핑이 가야한다. 해보자.

```bash
vagrant halt haproxy01

vagrant status
Current machine states:

minion1                   running (virtualbox)
minion2                   running (virtualbox)
minion3                   running (virtualbox)
haproxy01                 not created (virtualbox)
haproxy02                 running (virtualbox)
```

여전히 핑이 잘 간다.

vm을 다시 올리자.

```bash
vagrant up
```

## haproxy 설치

두개 모두 haproxy를 설치한다.

```bash
yum install -y haproxy
```

vi /etc/haproxy/haproxy.cfg

```bash
global
  daemon
  # 연결할 수 있는 최대 connection 을 지정한다. 이걸 안하면 기본값이 2000 으로 설정된다.
  maxconn     81920
  # haproxy 프로세서를 구동할 user (gid/uid를 지정할 수도 있다.)
  user        haproxy
  group       haproxy
  # haproxy는 로그를 남기기 위해서 file io 를 직접 처리하지 않는다. rsyslog 로 UDP 전송을 한다.
  log         127.0.0.1 local0
  # Number of processing cores. 기본값은 1
  nbproc      12
  chroot      /var/lib/haproxy
  pidfile     /var/run/haproxy.pid
  stats socket /var/lib/haproxy/stats

defaults
  log                     global
  mode                    http
  option                  httplog

  #Enable logging of null connections (헬스체크와 같이 시스템이 살아 있는지 확인하기 위해서 일정하게 접속하는 커넥션에 대한 로그 사용)
  option                  dontlognull
  #Enable skip logging normal connection log 일반적인 http status 200 로그는 남기지 않는다.
  #(https://www.slideshare.net/haproxytech/haproxy-best-practice) 참고
  option    dontlog-normal

  #request-요청을 서버로 보낼 때 `X-Forwarded-For` 를 헤더에 추가한다
  option forwardfor       except 127.0.0.0/8

  # 기본적으로 HAProxy 는 커넥션 유지 관점에서 keep-alive 모드로 동작을 하는데, 각각의 커넥션은 request-요청과 reponse-응답을 처리하고나서
  # 새로운 request을 받기까지 connection idle 상태(유휴상태)로 양쪽이 연결되어 있다.
  # 이 동작모드를 변경하려면 "option http-server-close" "option forceclose" "option httpclose" "option http-tunnel" 의 옵션이 가능한데,
  # "option http-server-close" 는 클라이언트 사이드에서 HTTP keep-alive를 유지하고 파이프라이닝을 지원하면서 서버 사이드에 커넥션을 닫는 형태를 설정한다.
  # 이는 클라이언트 사이드에서 최저 수준의 응답지연을 제공하고 "option forceclose" 와 비슷하게 서버사이드에서 리소스를 재활용할 수 있게 되어
  # backend 에서 빠르게 세션을 재사용할 수 있도록 해준다.
  option http-server-close

  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 6000

# haproxy 의 상태정보를 확인할 수 있는 기능을 활성화 한다. "stats"
# 이 포트는 외부에 노출되지 않도록 주의하자.
# 접근 가능한 사용자를 제한하기 위해서 auth 를 추가했다.
listen stats
  bind :8404 # Listen on localhost:9000
  stats enable  # Enable stats page
  stats realm Haproxy\ Statistics  # Title text for popup window
  stats uri /  # Stats URI
  stats auth UserName:Password
  stats refresh 5s

## 여기부터 필요한 내용으로 수정한다.
# frontend k8s-api
#     bind *:6443
#     mode tcp
#     option tcplog
#     timeout client 300000
#     default_backend k8s-api

# backend k8s-api
#     mode tcp
#     option tcplog
#     option tcp-check
# 	timeout server 300000
#     balance roundrobin
#     default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
#     server apiserver1 192.168.33.21:6443 check
#     server apiserver2 192.168.33.22:6443 check
#     server apiserver3 192.168.33.23:6443 check
```

http https를 리슨하고 있다가 192.168.33.10 으로 보내준다.

```bash
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
systemctl restart haproxy
```

http://192.168.33.10:8404/ 로 status 확인 가능

잘안되면 다음 커맨들 실행해보자.

```bash
setsebool -P haproxy_connect_any=1
#또는 selinux를 off하자.
```

웹사이트를 확인해보자.

## Enable Logs in CentOS

```
yum -y install rsyslog
vi /etc/rsyslog.conf
```

```bash
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
```

```
systemctl enable rsyslog
systemctl start rsyslog
systemctl status rsyslog
systemctl restart rsyslog
```

Verify the syslog server listening

```
netstat -antup | grep 514
```

로그확인

```
tail -f /var/log/messages
```

## 참고

<https://medium.com/@sliit.sk95/managing-failovers-with-keepalived-haproxy-c8de98d0c96e>
