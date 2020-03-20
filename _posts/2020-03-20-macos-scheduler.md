---
layout: post
title: 'macos scheduler' 
author: teamsmiley
date: 2020-03-20
tags: [macos]
image: /files/covers/blog.jpg
category: {macos}
---

# macos scheduler
macos는 리눅스와 다르게 cron을 쓰지 않는다.

launchd를 사용한다. 한참 헤매서 정리차 적어둔다.

목표 매시간마다 git을 업데이트 받고 싶다. 

리눅스면 크론으로 간단하게 하겠지만 맥에서 잘 안됬다. 

정리하면 간단하다.

shell을 먼저 만들자. 

vi _core_git_pull.sh
```bash
#! /bin/bash //꼭 사용
cd /Users/ragon/Desktop/GitHub/teamsmiley.github.io
git pull
```

~/Library/LaunchAgents/ 폴더 아래 다음 파일을 만든다. 

vi _git-update.plist
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>brian-gitpull</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>/Users/ragon/_core_git_pull.sh</string>
    </array>
    
    <key>RunAtLoad</key>
    <true/>
    
    <key>StartInterval</key>
    <integer>3600</integer>

    <key>StandardOutPath</key>
    <string>/tmp/brian.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/brian.log</string>

  </dict>
</plist>
```

이제 launchd에 적용하자.
```bash
launchctl load -w ~/Library/LaunchAgents/_backup.plist
launchctl load -w ~/Library/LaunchAgents/_git-update.plist

launchctl unload ~/Library/LaunchAgents/_backup.plist //삭제시
launchctl unload ~/Library/LaunchAgents/_git-update.plist//삭제시
```

`launchd list`를 하면 현재 돌고잇는 프로그램들을 볼수 있다.

여기서 주의할점
* 프로그램을 실행후 로그를 보고싶으면 tail -f /var/log/system 을 보면 된다.
* 프로그램이 처음에 되는지 안되는지 모르니 5초정도로 짧게해서 테스트를 하면 편한다.
* 로그 경로가 인터넷에 나온대로 /var/log/aaa.log를 하게 되면 프로그램이 아에 실행되지 않는다. 권한 문제 인듯. tmp는 잘됬음

이걸로 한달째 안되던 자동 백업과 업데이트가 완성 

## 추가 문제점

맥버전(catalina)에 따라서 Operation not permitted 이라는 에러가 나올수가 있다. 다음처럼 하자.

system preperences -> security and privacy -> privacy -> Full Disk Access 

Add > &#8984; + shift + G >> /bin/bash

![]({{ site_baseurl }}/assets/fulldiskaccess.png)





