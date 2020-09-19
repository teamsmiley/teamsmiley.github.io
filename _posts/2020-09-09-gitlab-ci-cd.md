---
layout: post
title: "ionic gitlab ci/cd"
author: teamsmiley
date: 2020-09-09
tags: [cicd]
image: /files/covers/blog.jpg
category: { cicd }
---

# 앱 개발이 마무리 되가서 ci/cd를 해야한다.

gitlab ci를 현재 사용중이므로 이걸 이용하기로 한다.

기존 프로젝트는 main/dev 브랜치에 머지 시점에 빌드 트리거를 작동햇는데 앱 빌드는 수동으로 클릭해서 사용해야한다.

일단 gitlab 서버는 설치되있다고 가정하고 gitlab runner를 설치해보자.

## xcode build

xcode에서 archive메뉴를 이용하면 업로드까지 모두 처리가 된다. 그러나 우리는 커맨드라인으로 처리를 해야한다. 그래야 gitlab runner가 실행해줄수 있다.

ExportOptions.plist 파일을 만들어야한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>method</key>
  <string>app-store</string>

  <key>teamID</key>
  <string>YOUR TEAMID</string>

  <key>signingCertificate</key>
  <string>YOUR CERT</string>

  <key>provisioningProfiles</key>
  <dict>
    <key>YOUR APP ID</key>
    <string>YOUR PROFILE NAME</string>
  </dict>

  <key>destination</key>
  <string>upload</string>
</dict>
</plist>
```

method로 app-store를 쓰겟다는것이고(ad hoc등을 사용할수도 있다)

teamid

![]({{ site_baseurl }}/assets/2020-09-19-08-46-23.png)

cert

![]({{ site_baseurl }}/assets/2020-09-19-08-41-45.png)

profile

![]({{ site_baseurl }}/assets/2020-09-19-08-42-53.png)

![]({{ site_baseurl }}/assets/2020-09-19-08-44-18.png)

이제 빌드하고 업로드해보자.

```bash
cd ./ios
xcodebuild -workspace 'App/App.xcworkspace' -scheme 'App' -configuration 'Release' -archivePath tmp.xcarchive archive # build and archive
xcodebuild -exportArchive -archivePath ./tmp.xcarchive -exportOptionsPlist ./ExportOptions.plist -exportPath ./exportIpaArchive/ # upload
```

업로드가 잘 된다. 이제 이걸 gitlab runner가 실행해주면된다.

## install gitlab-runner

on macos

```bash
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
cd ~
gitlab-runner install
gitlab-runner start
```

## register gitlab runner on macbook

```bash
sudo gitlab-runner register
```

![]({{ site_baseurl }}/assets/2020-09-09-13-10-43.png)

gitlab에서 admin > runner 에 가면 token 토큰을 찾을수 있다.

![]({{ site_baseurl }}/assets/2020-09-09-13-04-58.png)

gitlab 웹사이트에서 runner에 가서 등록이 됫는지 확인한다.

![]({{ site_baseurl }}/assets/2020-09-09-13-12-53.png)

추가 완료

러너를 눌러보면 restrict project for this runner라는 곳이 잇는데 여기에서 맥에서 빌드되야하는것만 선택한다.

![]({{ site_baseurl }}/assets/2020-09-09-13-14-46.png)

![]({{ site_baseurl }}/assets/2020-09-09-13-16-29.png)

specific 이라고 되면 성공

프로젝트에 가서 러너를 체크해보자.

project >> setting >> ci/cd >>

![]({{ site_baseurl }}/assets/2020-09-09-16-06-04.png)

나는 특정 프로젝트만 (ios빌드용) 사용할 예정이라 shared runner가 enable되잇으면 disable하자.

![]({{ site_baseurl }}/assets/2020-09-09-16-23-46.png)

완료

위처럼 수동으로 할수도 있고 자동화 스크립트로 처리 가능

```bash
gitlab-runner register \
 --non-interactive \
 --executor "shell" \
 --url "https://gitlab.your-domain.com/" \
 --registration-token "6y6A4circGrvrR" \
 --description "macbook-pro" \
 --tag-list "ios" \
 --run-untagged="true" \
 --locked="true"
```

참고로 설정은 ~/.gitlab-runner/config.toml 에 저장된다.

## gitlab-runner service start

```bash
gitlab-runner start
```

이제 빌드를 해보자.

vi .gitlab-ci.yml

```yml
stages:
  - build

build-dev:
  stage: build
  script:
    - echo "hello"
    - cd ./ios
    - xcodebuild -workspace 'App/App.xcworkspace' -scheme 'App' -configuration 'Release' -archivePath tmp.xcarchive archive
    - xcodebuild -exportArchive -archivePath ./tmp.xcarchive -exportOptionsPlist ./ExportOptions.plist -exportPath ./exportIpaArchive/
```

이제 커밋해보자. 푸시

자동으로 빌드가 시작되는지 확인해보자.

![]({{ site_baseurl }}/assets/2020-09-12-22-07-11.png)

![]({{ site_baseurl }}/assets/2020-09-12-22-08-56.png)

잘 빌드된다.

추가로 작업이 필요하면 스크립트를 만들어서 .gitlab-ci.yml에 추가하면 빌드가 되겟다.

## todo

ionic관련해서 빌드할 내용을 추가로 적어야할듯.

- mac에서 log보는법

- git pull을 해서 가져오는법.

- 특정 태그일때만 업로드하는법
- 버전 업데이트하는법
- 빌드 넘버 업데이트하는법
- 수동으로 트리거해서 빌드하는법
- test flight에 업로드가 되는데? 맞나?
- 정식버전으로 배포는 어떻게?
