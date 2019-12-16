---
layout: post
title: 'Macosx Terminal Setting' 
author: teamsmiley
date: 2018-08-01
tags: [git,terminal]
image: /files/covers/blog.jpg
category: {macosx}
---
# 맥 터미널 설정 

## homebrew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## vscode 설치 
```bash
brew cask install visual-studio-code 
```
실행후 command shift p >> path (옵션에 나오면 선택한다. `code` 명령어를 path에 등록한다는 이야기)

## zsh
mac이 가지고 있지만 (/bin/zsh) 다시 설치하자.

```bash
brew install zsh
which zsh
> /usr/local/bin/zsh

# standard shell로 추가
code /etc/shells
> /usr/local/bin/zsh # 추가

# 기본쉘로 등록
chsh -s /usr/local/bin/zsh
```

reboot

```bash
echo $SHELL
> /usr/local/bin/zsh
# check version
zsh --version
```

## oh my zsh
```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# update
upgrade_oh_my_zsh
```

## iterm2 설치
* install

```zsh
brew cask install iterm2
```

* Theme (Oceanic Next iTerm) <https://github.com/mhartington/oceanic-next-iterm> 

```
git clone https://github.com/mhartington/oceanic-next-iterm.git
```
다운로드 후 더블클릭 하면 설치됨

preference >> profile >> default >> color >> color presets >> oceanic-next-iterm
preference >> profile >> default >> text >> font >> hack

## zsh theme change
vi ~/.zshrc
```bash
ZSH_THEME="agnoster" #상단에 있는거 수정
```

```
source ~/.zshrc
```

## new line 
code ~/.oh-my-zsh/themes/agnoster.zsh-theme

```bash
## Main prompt
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  prompt_aws
  prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_newline  # 여기 추가
  prompt_end
}
{% raw %}
prompt_newline() {
  if [[ -n $CURRENT_BG ]]; then
    echo -n "%{%k%F{$CURRENT_BG}%}$SEGMENT_SEPARATOR
%{%k%F{blue}%}$SEGMENT_SEPARATOR"
  else
    echo -n "%{%k%}"
  fi

  echo -n "%{%f%}"
  CURRENT_BG=''
}
{% endraw %}
```

## host명 지우기 
```bash
echo "prompt_context() {}" >> ~/.zshrc
```

## oh my zsh plugin 
리스트 확인하기 

`ls ~/.oh-my-zsh/plugins`

현재는 git만 활성화 되잇는것을 알수 있다. 추가를 원하는 플러그인 이름을 적어주면된다.

vi ~/.zshrc
```
plugins=(git)
```

```bash
# Syntax Hightlight
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
# autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

ls ~/.oh-my-zsh/custom/plugins
```

vi ~/.zshrc
```
plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
```

## 추가 program 들
```bash
brew cask install github
brew cask install tunnelblick
brew cask install sequel-pro
# 최신버전
brew cask install dotnet-sdk
# 구버전
brew tap isen-ng/dotnet-sdk-versions
brew cask install dotnet-sdk-2.2.400
# 확인
dotnet --list-sdks
brew cask install microsoft-azure-storage-explorer
brew cask install docker
brew cask install postman
brew cask install grammarly
brew cask install firefox
brew cask install slack
```


