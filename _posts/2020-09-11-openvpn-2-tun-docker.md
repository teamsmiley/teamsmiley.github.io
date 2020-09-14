---
layout: post
title: "Openvpn-TUN-docker"
author: teamsmiley
date: 2020-09-11
tags: [openvpn]
image: /files/covers/blog.jpg
category: { network }
---

# openvpn - TUN - docker

사내 vpn을 구축해서 사용해야해서 openvpn을 이용해서 구축을 해 보았다.

vpn을 구축할때는 TUN (layer 3 , nat) 방식과 TAP (layer 2 , bridge) 방식이 있다.

TUN 방식 : 일반적으로 공유기를 쓰는것과 비슷하다. 사내에서 사용하는 아이피 (예 192.168.0.xxx) 를 사용하지 않고 새로운 대역을 할당받아서 vpn이 서로 연결해주는 방식이다.

TAP 방식 : 일반적으로 사내에 switch를 하나더 연결한 것과 같고 사내에서 사용하는 아이피 (예 192.168.0.xxx)와 같은 대역의 아이피를 할당받는다.

## server setup

centos 7 로 진행해보고 방화벽은 끄고 하던지 아니면 다음 세팅을 해주면 될듯 싶다.

### firewalld centos 7

```bash
firewall-cmd –-add-service openvpn –-permanent
firewall-cmd –-add-masquerade –-permanent

firewall-cmd --reload
systemctl restart firewalld

# or
# systemctl stop firewalld
# systemctl disable firewalld
```

### ip forward

```bash
vi /etc/sysctl.conf

net.ipv4.ip_forward = 1
```

```bash
systemctl restart network.service
```

vpn에서 받은걸 ip forward 를 통해서 전달해줘야한다.

## TUN 방식 (docker)

기본적인 설명은 앞글에서 다 설명을 햇으므로 확인하시길

### install

```bash
ssh vpn01

OVPN_DATA="/data/git/docker/openvpn/config"

# init
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://---.---.---.--- # 본인 아이피를 넣으세요
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
> yourpassword
> server name
wait
> yourpassword
> yourpassword

# start openvpn
docker run --name openvpn -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --privileged --restart unless-stopped kylemanna/openvpn

# 확인
docker ps -a
```

## 유저 추가

```bash
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass # client key를 만드는 과정
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_getclient CLIENTNAME > $OVPN_DATA/CLIENTNAME.ovpn # 만들어진 키들을 하나의 ovpn파일로 만드는 과정
```

docker 실행시 경로를 매핑해서 사용했기때문에 \$OVPN_DATA/CLIENTNAME.ovpn 에 파일이 생김

```bash
scp vpn01:/data/git/docker/openvpn/config/CLIENTNAME.ovpn .
```

scp 로 가져와서 사용 클라이언트에서 사용

\$OVPN_DATA 경로에 가면 설정파일이 생겻을텐데 다음처럼 변경한다.

## openvpn 서버 설정

```bash
cd /data/git/docker/openvpn/config
vi openvpn.conf

server 192.168.255.0 255.255.255.0
verb 3
key /etc/openvpn/pki/private/204.16.116.121.key
ca /etc/openvpn/pki/ca.crt
cert /etc/openvpn/pki/issued/204.16.116.121.crt
dh /etc/openvpn/pki/dh.pem
tls-auth /etc/openvpn/pki/ta.key
key-direction 0
keepalive 10 60
persist-key
persist-tun

proto udp
# Rely on Docker to do port mapping, internally always 1194
port 1194
dev tun0
status /tmp/openvpn-status.log

user nobody
group nogroup
comp-lzo no

### Route Configurations Below
route 192.168.254.0 255.255.255.0

### Push Configurations Below
push "block-outside-dns"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
push "comp-lzo no"

### 추가

# 중복커넥션 허용
duplicate-cn
push "route 192.168.0.0 255.255.255.0"

# push "redirect-gateway def1 bypass-dhcp" 이걸 주석풀면 모든 트래픽이 서버로 간다. 인터넷도 vpn을 통해서 된다. (서버에서 강제시 사용)
```

도커를 재시작한다.

`docker restart openvpn`

## client 설정

ovpn파일에서 정한다.

- 모든 트래픽을 vpn으로 보낼경우
  ovpn 설정 마지막 이부분을 적는다.

```bash
redirect-gateway def1
```

- 일반적인 인터넷을 사용하고 routing에 추가가된 서브넷만 vpn으로 보낸다.
  주석처리한다.

```bash
# redirect-gateway def1
```

## docker-compose로 변경

매번 도커를 재시작하기 귀찮아서 compose를 이용한다.

vi /data/git/docker/openvpn/docker-compose.yml

```yml
---
version: "3.3"
services:
  openvpn:
    container_name: "openvpn"
    image: "kylemanna/openvpn"
    restart: always
    hostname: "openvpn"
    environment:
      TZ: "America/Los_Angeles"
    ports:
      - "1194:1194/udp"
    privileged: true
    volumes:
      - "/data/git/docker/openvpn/config:/etc/openvpn"
```

`docker-compose up -d`로 사용하면된다.
