---
layout: post
title: "Openvpn-TUN"
author: teamsmiley
date: 2020-09-11
tags: [openvpn]
image: /files/covers/blog.jpg
category: { network }
---

4년전부터 대충 사용하던 openvpn의 찜찜한 부분을 공부해서 다시 정리해보았습니다. 한글 자료가 많이 부족하여 직접 작성하게 되었습니다. 연제 글로 올려봅니다.

1. <https://teamsmiley.github.io/2020/09/11/openvpn-1-tun/>
1. <https://teamsmiley.github.io/2020/09/11/openvpn-2-tun-docker/>
1. <https://teamsmiley.github.io/2020/09/11/openvpn-3-tap/>

# openvpn - TUN

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

## TUN 방식

### install

```bash
ssh vpn01
yum install epel-release -y
yum update -y
yum install -y openvpn wget
```

- easy-rsa 2

easy-rsa 2 가 필요하다 그런데 3도 있기는한데 대충 분위기가 2.xx를 쓰는 분위기 openvpn도 3 이 있으나 2.x 쓰는 분위기

혹시 모르니 버전번호도 체크해보기 바란다.

<https://github.com/OpenVPN/easy-rsa-old/releases>

```bash
cd
wget -O /tmp/easyrsa https://github.com/OpenVPN/easy-rsa-old/archive/2.3.3.tar.gz
tar xfz /tmp/easyrsa
mkdir -p /etc/openvpn/easy-rsa
cp -rf easy-rsa-old-2.3.3/easy-rsa/2.0/* /etc/openvpn/easy-rsa
ls -alF /etc/openvpn/easy-rsa
```

### Configuring OpenVPN

```bash
cp /usr/share/doc/openvpn-2.4.9/sample/sample-config-files/server.conf /etc/openvpn
vi /etc/openvpn/server.conf
```

기본 설정을 설명

```bash
port 1194

# TCP or UDP server?
;proto tcp
proto udp

# tap or tun tun은 nat을 통해서 트래픽을 전달하고 tap은 layer 2 로 브릿지로 사용한다.
;dev tap
dev tun

ca ca.crt
cert server.crt
key server.key  # This file should be kept secret

dh dh2048.pem

server 10.8.0.0 255.255.255.0 # vpn과 클라이언트 사이에 사용할 ip 대역을 표시한다.

ifconfig-pool-persist ipp.txt

;push "route 192.168.10.0 255.255.255.0" # 주석 제거하고 사용하는 ip 를 넣는다.
push "route 192.168.0.0 255.255.255.0" # 현재는 192.168.0.xxx대역에 vpn 클라이언트가 연결하고 싶다

keepalive 10 120

tls-auth ta.key 0

# Select a cryptographic cipher.
# This config item must be copied to
# the client config file as well.
# Note that v2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the ncp-cipher option in the manpage
;cipher AES-256-CBC # 2.4 이상은 AES-256-GCM로 자동으로 한다고함. 주석처리하자.

persist-key
persist-tun

status openvpn-status.log
```

기본 설정에서 다음코드만 추가한 파일이였음

```bash
push "route 192.168.0.0 255.255.255.0"
;cipher AES-256-CBC
user nobody
group nobody

push "dhcp-option DNS 8.8.8.8"
```

### Generating Keys and Certificates

```bash
mkdir /etc/openvpn/easy-rsa/keys
vi /etc/openvpn/easy-rsa/vars

export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="LosAngeles"
export KEY_ORG="XGrid"
export KEY_EMAIL="teamsmiley@gmail.com"
export KEY_EMAIL=teamsmiley@gmail.com
export KEY_CN=vpn01
export KEY_NAME=vpn01-server
export KEY_OU=devteam
export PKCS11_MODULE_PATH=changeme
export PKCS11_PIN=1234
```

필요한 내용은 변경하고 pkcs관련은 그대로 둔다. keycard나 동글 등을 이용할때 사용하는변수라고함. 사용하지 않으므로 그대로 두고 진행

이제 적용해보자.

```bash
cd /etc/openvpn/easy-rsa
source ./vars
ls -alF /etc/openvpn/easy-rsa/keys
./clean-all # key를 다 지운다.
ls -alF /etc/openvpn/easy-rsa/keys # 지워졋는지 확인
./build-ca
```

아까 설정해준값이 기본값으로 나온다. 엔터만 치면됨.

```bash
./build-key-server server
```

또 엔터만 치면된다. 그런데 갑자기 extra를 물어본다. 그냥 엔터

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

```
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :PRINTABLE:'CA'
localityName          :PRINTABLE:'LosAngeles'
organizationName      :PRINTABLE:'Yourcompany'
organizationalUnitName:PRINTABLE:'devteam'
commonName            :PRINTABLE:'server'
name                  :PRINTABLE:'vpn01-server'
emailAddress          :IA5STRING:'teamsmiley@gmail.com'
Certificate is to be certified until Sep  8 15:26:51 2030 GMT (3650 days)
Sign the certificate? [y/n]:y
```

y를 누르고 진행한다. 10년짜리 cert가 만들어 진다.

Diffie-Hellman key exchange file 을 만들어야함.

```bash
./build-dh  # This is going to take a long time
ls -alF /etc/openvpn/easy-rsa/keys  # 확인
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn # 세팅파일에 있던 설정대로 복사해준다.
ls -alF /etc/openvpn
```

이제 openvpn server는 다 되었다.

### client key를 만들어서 사용할수 있게 하자.

```bash
cd /etc/openvpn/easy-rsa
./build-key ragon
```

```
Common Name (eg, your name or your server's hostname) [ragon]: ragon
Name [vpn01-server]:ragon
Email Address [support@xgridcolo.com]:teamsmiley@gmail.com
```

3개의 값에 주의하여 입력하자.

### setup openssl.cnf

```bash
cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
```

### routing

vpn에 연결되면 트래픽이 vpn까지는 오는데 그 트래픽을 외부로 보내줄 설정을 해야한다. iptables를 이용한다.

```bash
NIC_NAME=$(ip route get 8.8.8.8 | awk 'NR==1 {print $(NF-2)}')
echo $NIC_NAME
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o $NIC_NAME -j MASQUERADE
```

### Starting OpenVPN

```bash
systemctl -f enable openvpn@server.service
systemctl start openvpn@server.service
systemctl status openvpn@server.service
systemctl restart openvpn@server.service
```

### Client로 접속 테스트

```bash
cd
scp vpn01:/etc/openvpn/easy-rsa/keys/ragon.crt ./Desktop/openvpn/
scp vpn01:/etc/openvpn/easy-rsa/keys/ragon.key ./Desktop/openvpn/
scp vpn01:/etc/openvpn/ca.crt ./Desktop/openvpn/
scp vpn01:/etc/openvpn/ta.key ./Desktop/openvpn/
```

vi ragon.ovpn

```bash
client
nobind
dev tun

proto udp

remote ---.---.---.--- 1194 udp #본인 아이피를 넣으세요

key-direction 1

cert /Users/ragon/Desktop/openvpn/ragon.crt
key /Users/ragon/Desktop/openvpn/ragon.key

ca /Users/ragon/Desktop/openvpn/ca.crt
tls-auth /Users/ragon/Desktop/openvpn/ta.key

;push "redirect-gateway def" #전체 트래픽을 vpn으로 보내고싶을때 사용
```

ifconfig를 보면

```
utun2: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1500
	inet 10.8.0.6 --> 10.8.0.5 netmask 0xffffffff
```

6번 아이피를 받앗고 5번으로 보내주는걸 알수 있다. tun모드이다. nat를 해서 전달중

ping 192.168.0.1번을 laptop에서 핑 되는지 확인하자.

### 서버 설정 업데이트

```bash
port 11194        # 기본포트 변경
client-to-client  # 클라이언트끼리 연결
duplicate-cn      # 하나의 키로 중복 연결 허용
user nobody       # 권한
group nobody      # 권한
```

- 하나의 키로 여러 컴에서 접속 가능한지 확인. duplicate-cn

서버에 세팅을 해서 동작해야한다.

테스트하자. 잘 된다.

하나의 컴퓨터는 10.8.0.6(pc) --> 10.8.0.5(vpn)

다른 하나는 10.8.0.14(pc)--> 10.8.0.15 (vpn)

### 클라이언트 설정 업데이트

- port 변경

```bash
remote ---.---.---.--- 11194 udp # 본인 아이피를 넣으세요
```

포트 변경하고 다시 설치하고 접속하면 잘된다.

- 1개의 파일로 다 관리하기

파일 경로로 ovpn을 사용하지 않고 하나의 파일에 전부 넣는 방법도 있다.

참조 파일을 열어서 내용을 접속 파일에 넣어주면 된다.

```bash
client
nobind
dev tun

proto udp

remote ---.---.---.--- 11194 udp # 본인 아이피를 넣으세요

key-direction 1

# cert /Users/ragon/Desktop/openvpn/ragon.crt
# key /Users/ragon/Desktop/openvpn/ragon.key

# ca /Users/ragon/Desktop/openvpn/ca.crt
# tls-auth /Users/ragon/Desktop/openvpn/ta.key

# push "redirect-gateway def"

<key>
-----BEGIN PRIVATE KEY-----
MIIJQgIBADANBgkqhkiG9w0BAQEFAASCCSwwggkoAgEAAoICAQDoB9DdGMRmrN
-----END PRIVATE KEY-----
</key>

<cert>
-----BEGIN CERTIFICATE-----
MIIHCDCCBPCgAwIBAgIBAjANBgkqhkiG9w0BAQUFADCBnjELMAkGA1UEBhMCVVMx
wbvKqoc8sGQ8m0wHL5kOAmEIg8d5nqlHmlHWct3wM9HgwC71SSAubZrWY9gnMZOl
-----END CERTIFICATE-----
</cert>

<ca>
-----BEGIN CERTIFICATE-----
MIIGyjCCBLKgAwIBAgIJAPPPBQknJ1VIMA0GCSqGSIb3DQEBCwUAMIGeMQswCQYD
VQQGEwJVUzELMAkGA1UECBMCQ0ExEzARBgNVBAcTCkxvc0FuZ2VsZXMxDjAMBgNV
</ca>

<tls-auth>
-----BEGIN OpenVPN Static key V1-----
c68d95426c005f2ecf85a0f062a404ff
12f999589c8918905a6ac46ddfbe8ffd
</tls-auth>
```

이렇게 하면 하나의 파일로 관련정보를 모두 관리할수 있어 편하다.

### topology subnet & client to client

현재는 네트워크를 /30로 잘라서 (4개의 아이피만 사용가능) vpn과 클라이언트끼리 연결하기 때문에 다른 클라이언트와 서로 연결이 되지 않는다.

A => vpn <= B

이런 구조인데 A와 B가 연결되지는 않는다.

만약 필요하다면 client to client를 세팅하면된다.

서버 설정에서 주석 제거후 재시작

```bash
;topology subnet
;client to client
```

클라이언트 접속하면

```
utun2: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1500
inet 10.8.0.3 --> 10.8.0.3 netmask 0xffffff00
```

3번 할당

다른 pc에서는 4번 할당

서로 핑이 될가? 된다.
