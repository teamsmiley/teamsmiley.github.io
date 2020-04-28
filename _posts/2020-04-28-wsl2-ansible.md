---
layout: post
title: 'Wsl2 Ansible' 
author: teamsmiley
date: 2020-04-28
tags: [kube]
image: /files/covers/blog.jpg
category: {kube}
---

현재 ansible을 사용하기 위해 리눅스 서버에 접속하여 ansible-playbook명령어를 실행하고 있다. 

그런데 wsl2를 사용하면 가능할듯 보여 시도 해보았다.

ubuntu 2004 distro를 사용한다. ubuntu를 root로 접속해서 사용중이다. 관련 내용은 기존 포스트에 있다.

```bash
apt-get update
apt install ansible -y
ansible --version
> ansible 2.9.6
```

ubuntu 1804에서는 
```bash
apt-get update
apt-get -y install python-pip python-dev libffi-dev libssl-dev
pip install ansible
ansible --version
```

이제 vpn을 연결후 ping을 해보자. 

```bash
ansible host01 -m ping
```

Error 

```bash
[WARNING]: Ansible is being run in a world writable directory
(/mnt/c/Users/ragon/Desktop/GitLab/xgrid-server/acs01/acs/xgrid), ignoring it as an ansible.cfg source. For more information see https://docs.ansible.com/ansible/devel/reference_appendices/config.html#cfg-in-world-writable-dir
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
[WARNING]: Could not match supplied host pattern, ignoring: host01
```

해결
```
export ANSIBLE_CONFIG=./ansible.cfg
```

잘된다.

## alias 만들기 

acs를 치면 다음 명령어가 실행되면 좋겟다.
vi ~/.zshrc

```bash
alias acs="cd /mnt/c/Users/ragon/Desktop/GitLab/xgrid-server/acs01/acs/xgrid"
```

이제 acs만 치고 `ansible host01 -m ping` 하면된다.






