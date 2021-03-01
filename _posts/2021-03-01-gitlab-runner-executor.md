---
layout: post
title: "gitlab tip - executor, tag, release"
author: teamsmiley
date: 2021-03-01
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# gitlab runner executor

gitlab runner executor 는 여러가지가 있다. <https://docs.gitlab.com/runner/executors/>

기본적인 shell과 docker만 다루어 보겟다. 하나의 장비에서 두개의 runner를 돌려서 tag가 docker가 붙으면 도커 executor가 돌게 해보자.

## executor 등록

### shell executor

```bash
ssh runner01

gitlab-runner register \
 --non-interactive \
 --executor "shell" \
 --url "https://gitlab.yourdomain.com/" \
 --registration-token "xxxNM11xxKvg6TDKtxs3" \
 --description "runner01" \
 --tag-list "" \
 --run-untagged="true" \
 --locked="true"
```

### docker executor

```bash
 gitlab-runner register \
 --non-interactive \
 --executor "docker" \
 --docker-image alpine:latest \
 --url "https://gitlab.yourdomain.com/" \
 --registration-token "xxxM11xxKvg6TDKtxs3" \
 --description "runner01-docker-executor" \
 --tag-list "docker" \
 --run-untagged="true" \
 --locked="true"
```

등록하고 나면 다음 파일에 내용이 정리되는걸 볼수가 있다.

```bash
cat /etc/gitlab-runner/config.toml

concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "runner01"
  url = "https://gitlab.yourdomain.com/"
  token = "xxxpzshVpBoXzz4XA6y"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]

[[runners]]
  name = "runner01-docker-executor"
  url = "https://gitlab.yourdomain.com/"
  token = "xxx8XwGwqt4T39ahqu"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
```

### 참고

config.toml파일을 수정하면 자동 적용된다. runner를 재시작 할 필요가 없다는것이다.

## tags 사용하기

위에서 runner를만들때 컴마로 구분하여 자신이 받을 태그를 적어놓는다.

깃랩이 작업을 던질 때 이 태그가 있는 러너에만 작업을 할당한다.

중요한것은 태그가 여러개 있으면 모든 태그를 다 가지고 있는 러너가 실행이 된다.

## Gitlab Release 만들기

Gitlab에서 요즘 업데이트로 인해서 Release도 편하게 만들수 잇게 되있다.

### shell executor

```yml
stages:
  - release

release:
  stage: release
  when: manual
  script:
    - >
      release-cli create --name release-$CI_JOB_ID --description release-$CI_COMMIT_REF_NAME-$CI_JOB_ID --tag-name job-$CI_JOB_ID --ref $CI_COMMIT_SHA
```

shell executor는 release-cli를 따로 설치를 해줘야한다.

```bash
curl -L --output /usr/local/bin/release-cli "https://release-cli-downloads.s3.amazonaws.com/latest/release-cli-linux-amd64"
sudo chmod +x /usr/local/bin/release-cli
```

### docker executor

```yml
stages:
  - release

Release-Node:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - echo $CI_COMMIT_TAG
  release:
    name: release-$CI_COMMIT_TAG
    description: release-$CI_COMMIT_TAG
    tag_name: $CI_COMMIT_TAG
    ref: $CI_COMMIT_TAG
  tags:
    - docker
```

tags에 docker를 추가하여 이 태그가 있는 러너에서만 실행이 되게 해두었다.

실행해보면 잘된다.

### 참고

release를 지우는것은 ui에는 없으나 tag자체를 지우면 release가 지워지는것을 확인할수 있다.
