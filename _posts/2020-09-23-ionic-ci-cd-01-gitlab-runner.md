---
layout: post
title: "ionic ci/cd - 01 gitlab runner"
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

# ionic ci/cd - gitlab runner

gitlab ci를 현재 사용중이므로 이걸 이용하기로 한다.

일단 gitlab 서버는 설치되있다고 가정하고 macbook pro에 gitlab runner를 설치 하고 실행하면 된다.

## gitlab-runner

### install gitlab-runner

on macos

```bash
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
cd ~
gitlab-runner install
gitlab-runner start
```

### register gitlab runner on macbook

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

### gitlab-runner service start

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
```

이제 커밋해보자. 푸시

자동으로 빌드가 시작되는지 확인해보자.

![]({{ site_baseurl }}/assets/2020-09-12-22-07-11.png)

![]({{ site_baseurl }}/assets/2020-09-12-22-08-56.png)

잘 빌드된다.

추가로 작업이 필요하면 스크립트를 만들어서 .gitlab-ci.yml에 추가하면 빌드가 되겟다.

### log 확인

application >> util >> console : filter by gitlab

로그가 보이면서 에러내용도 볼수 있다.

## ci/cd trigger

### master에 푸시할때만 ci/cd를 실행

```yml
stages:
  - build

build-staging:
  stage: build

  script:
    - echo "hello"
  only:
    - master
```

마스터일때만 ci/cd를 실행한다.

### 태그를 푸시할때만 ci/cd를 실행하고싶다.

생각해보니 마스터일때만 할게 아니라 버전을 태깅을 하면 그때 동작하게 하고 싶다.

yml에 tags를 추가한다.

```yml
stages:
  - build

build-staging:
  stage: build
  script:
    - echo "hello"
  only:
    - tags
```

이제 commit / push를 하고 난후 tagging을 한다.

```bash
git tag -m "release v1.0.0" -a v1.0.0 05d9873 # git hash
git push origin v1.0.0                        # push tags
```

1.0.0 태깅을 하면 ci/cd가 트리거 된다.

## known error

### gitlab runner stuck

혹시 gitlab runner가 stuck상태이면 runner 상태를 보자.

admin >> runner >> 특정 러너 수정

![]({{ site_baseurl }}/assets/2020-09-21-11-45-56.png)

indicate whether this runner can pick jobs without tags => on

아래쪽 tags는 빈칸으로 해둬야한다.
