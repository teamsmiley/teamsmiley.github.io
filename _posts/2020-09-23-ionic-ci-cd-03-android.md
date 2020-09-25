---
layout: post
title: "ionic ci/cd - 03 android"
author: teamsmiley
date: 2020-09-23
tags: [cicd]
image: /files/covers/blog.jpg
category: { ionic }
---

얼마전 쓴 글이 정리도 잘 안되고 너무 커서 자르고 정리해서 다시 올립니다.

1. <https://teamsmiley.github.io/2020/09/23/ionic-ci-cd-01-gitlab-runner/>
1. <https://teamsmiley.github.io/2020/09/23/ionic-ci-cd-02-ios/>
1. <https://teamsmiley.github.io/2020/09/23/ionic-ci-cd-03-android/>

# ionic ci/cd - android

## android studio install

<https://developer.android.com/studio>

## command line으로 빌드/업로드

```bash
cd project directory

ionic build --configuration=production && npx cap copy android && npx cap update android && npx cap open android

cd android

gradlew tasks # 전체 gradle tasks
bash gradlew #build

# apk bundle 생성(unsigned)
bash gradlew :app:bundleDebug

## sign

## upload
```

갑자기 자바를 설치하라고함.

https://www.oracle.com/java/technologies/javase-downloads.html

11버전이 lts니 그걸로 macOS Installer 를 설치

그런데 더 조사를 해보니 gradlew를 쓰면 필요없어 보임

```
cd android
chmod +x gradlew

./gradlew tasks --all
```

apksigner sign --ks release.jks app.apk

Sign your app from command line

keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias

## ci/cd 파일로 스크립트 실행

## known error
