---
layout: post
title: 'cloud-init'
author: teamsmiley
date: 2021-02-08
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# cloud-init

cloud-init은 다양한 리눅스를 vm에 설치시 vm이 처음 로딩된후 실행되는 것 같다.

SSH 액세스 키, 유저 생성, 특별한 패키지 설치 등등 할수잇는일이 아주 많습니다.

왜 필요하냐면 기존에는 vm만들고 올라오면 console로 ip설정하고 ip로 접근해서 이런저런것을 해서 서버로 준비후 서비스에 넣게 되는데 auto provision을 사용하게 되면 vm이 생성과 동시에 ip받고 자체적으로 다 설정하고 재부팅하고 나면 서비스에 바로 적용됩니다. 결론은 vm초기 설치시부터 서비스 적용까지 사람의 손을 안타려고 만든거같습니다.

거의 모든 리눅스가 cloud init을 지원합니다.

아마존 / ms / 구글 클라우드회사도 모두 지원합니다.

cloud-init은 보통 boot 시점에 발생하며, 5 가지의 단계를 거칩니다.

## 단계

총 5단계가 있습니다.

- Generator
- Local
- Network
- Config
- Final

## maas 에서 사용하기

deploy하기전에 다음 화면에 내용을 넣어두면 설치시 실행을 해줍니다.

![]({{ site_baseurl }}/assets/2021-02-09-12-07-02.png)

```yml
#cloud-config
users:
  - default
  - name: YourUserName
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    groups: sudo
    lock-passwd: false
    passwd: hashed_your_password
timezone: America/Los_Angeles
packages:
  - htop
  - tree
  - net-tools
  - zsh
package_update: true
package_upgrade: true
manage_etc_hosts: false
runcmd:
  - su ubuntu -c 'sh -c "$(curl -fsSL https://raw.githubusercontent.com/coreycole/oh-my-zsh/master/tools/install.sh)"'
  - chsh -s $(which zsh) ubuntu
  - sed -i "s/  git/  git\n  kubectl\n  kube-ps1\n  zsh-syntax-highlighting\n  zsh-autosuggestions/g" /home/ubuntu/.zshrc
  - echo "alias watch='watch ' " >> /home/ubuntu/.zshrc
  - echo "NEWLINE=\$'\\\\n'" >> /home/ubuntu/.zshrc
  - echo "export PROMPT='[\$FG[154]%T%{\$reset_color%}][%{\$fg[cyan]%}%m %{\$reset_color%}%~] \$(git_prompt_info)\${NEWLINE}# '" >> /home/ubuntu/.zshrc
  - echo "bindkey -v" >> /home/ubuntu/.zshrc
  - cd /home/ubuntu/.oh-my-zsh/custom/plugins/
  - git clone https://github.com/zsh-users/zsh-autosuggestions
  - git clone https://github.com/zsh-users/zsh-syntax-highlighting
  - sudo chmod 755 -R .
power_state:
  delay: '2'
  mode: reboot
  message: Bye Bye
  timeout: 5
  condition: True
```

`#cloud-config` 이건 꼭 써주셔야 합니다.

### 위에서 한일

- 가끔 vm에 문제가 잇을때 콘솔로 접속해야하는데 그 때 사용할 유저를 하나 만들고
- timezone설정
- 추가 packages 설치
- apt update 실행
- apt upgrade 실행
- 재부팅시마다 hosts 파일이 초기화 되는데 이걸 하지 /etc/hosts를 cloud-init이 관리하지 않겟다고 설정
- oh-my-zsh을 설치하고 관련 내용을 설정 프롬프트도 설정
- 모두 완료되고 나면 서버를 reboot한다. poweroff로도 할수 있다.

이제 vm이 올라오면서 이작업을 다 해주니 몇대를 올리더라도 부담이 없다.

## 추가

추가 필요한 내용은 다음에서 확인해보기 바란다.

<https://cloudinit.readthedocs.io/en/latest/topics/examples.html>
