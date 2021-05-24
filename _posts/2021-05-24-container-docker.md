---
layout: post
title: 'container - docker'
author: teamsmiley
date: 2021-05-24
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# container - docker

## 가장 중요한 내용

도커란 실행에 필요한 모든 내용을 가지고 있는 프로세스다. 프로세스.

Immutable Infrastructure를 구축해서 사용하는것이 도커(컨테이너)방식이다. 기존방식에서 벗어나야한다.

자바를 배워서 c처럼 코딩하면 안되듯이 새로운 철학으로 생각을 바꾸는것이 제일 중요. 그러나 제일 어렵다는건 함정.

Immutable Infrastructure는 관리가 편해지고 확장이 쉬워지며 운영체제와 서비스운영 환경이 분리되서 가볍고 어디서나 실행가능해진다. 심지어는 오랜 기간이 지나도 실행이 가능해진다.

컨테이너: 필요한 모든 내용을 하나의 상자에 넣어두고 상자를 포장해둔 것이다.

이제 이 상자만 여기저기 가지고 다니더라도 어디서든 실행되고 관리가 편하게 된다.

이 상자를 이미지로 만들면 도커 이미지가 된다.

도커 이미지를 실행한 상태를 컨테이너라고 한다. 자바에서 클래스 오브젝트 인스턴스 와 비슷

## search 명령어

```bash
docker search  # 가능하면 docker hub에서 찾는게 더 편할때가 많다.
```

docker hub - <https://hub.docker.com/search?q=nginx&type=image>

## 이미지 다운로드

```bash
docker pull ubuntu:latest #공식
docker pull myprivateserver:5000/ubuntu:latest #프라이빗 서버도메인과 포트를 이름에 포함하는게 중요
```

## 이미지 관련 명령어

```bash
docker image --help

docker image build       # Build an image from a Dockerfile
docker image history     # Show the history of an image
docker image import      # Import the contents from a tarball to create a docker image filesystem image
docker image inspect     # Display detailed information on one or more images
docker image load        # Load an image from a tar archive or STDIN
docker image ls          # List images
docker image prune       # Remove unused images
docker image pull        # Pull an image or a repository from a registry
docker image push        # Push an image or a repository to a registry
docker image rm          # Remove one or more images
docker image save        # Save one or more images to a tar archive docker image (streamed to STDOUT by default)
docker image tag         # Create a tag TARGET_IMAGE that refers to docker image SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```

```bash
docker image rm --help

Usage:  docker image rm [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Aliases:
  rm, rmi, remove

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

## run 명령어

```bash
docker run --help # 관련 내용을 다 볼수 있다.

docker run -i -t --name test-name ubuntu /bin/bash # 우분투를 실행해서 bash를 실행해라
```

-i(interactive), -t(Pseudo-tty) 옵션을 사용하면 실행된 Bash 셸에 입력 및 출력을 할 수 있습니다.

--name 옵션으로 컨테이너의 이름을 지정. 이름을 지정하지 않으면 Docker가 자동으로 이름을 생성하여 지정

## docker ps

```bash
docker ps

docker ps --help

Options:
  -a, --all             # Show all containers (default shows just running)
  -f, --filter filter   # Filter output based on conditions provided
      --format string   # Pretty-print containers using a Go template
  -n, --last int        # Show n last created containers (includes all states) (default -1)
  -l, --latest          # Show the latest created container (includes all states)
      --no-trunc        # Don't truncate output
  -q, --quiet           # Only display container IDs 이거 많이 사용
  -s, --size            # Display total file sizes
```

docker ps 는 실행되는 프로세스만 보여주고 -a까지 붙이면 stop되잇는 모든 프로세스가 보인다.

## docker status

```bash
docker start test #container name , 또는 컨테이너 아이디 첫 4자만 쓰면된다.
docker stop test
docker start test
docker restart test
docker attach test
```

container name으로 사용이 가능, 또는 컨테이너 아이디 첫 4자 사용가능 물론 전체 아이디 사용가능

container 실행중 Bash 셸에서 exit 또는 Ctrl+D를 입력하면 컨테이너가 정지
container 실행중 Bash 셸에서 Ctrl+P, Ctrl+Q를 차례대로 입력하여 컨테이너를 정지하지 않고, 컨테이너에서 exit

## Dockerfile

docker 이미지를 만들수 있는 파일을 만든다. 이것이 개발자들은 제일 중요

```yml
FROM ubuntu:20.04

RUN apt-get update
RUN apt-get install -y nginx

CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80
```

- RUN
  RUN은 FROM에서 설정한 이미지 위에서 스크립트 혹은 명령을 실행합니다

- CMD
  CMD는 컨테이너가 시작되었을 때 스크립트 혹은 명령을 실행, /bin/sh 실행 파일을 사용

- ENTRYPOINT
  ENTRYPOINT는 컨테이너가 시작되었을 때 스크립트 혹은 명령을 실행합니다.

- EXPOSE
  호스트와 연결할 포트 번호입니다.

- VOLUME:
  호스트와 공유할 디렉터리 목록입니다. docker volume ls로 확인되는것을 사용하거나 절대경로를 사용해야한다. 이름만 적으면 docker volume을 미리 만들어 두어야한다.

전체 옵션을 다음을 확인 <https://docs.docker.com/engine/reference/builder/>

## build image

Dockerfile이 만들어졌으므로 이제 빌드를 해보자.

```bash
docker build --tag test .    # version을 안붙이면 latest가 자동으로 붙는다.
docker build --tag test:latest .
docker build --tag test:0.1 . # 버전을 :이후에 붙여주면 좋다.
docker build --tag your-private-registry:5000/test:0.1 # 도커허브가 아닌 따로 레지스트리를 사용한 경우
```

마지막에 `.` 을 꼭 잊지말기 바란다. Dockerfile이 있는 폴더에서 실행해야하며 현재 폴더를 나타내는 `.` 을 꼭 사용해야한다.

가능하면 latest는 쓰지 않는걸 추천드린다. 항상 버전을 기입하고 사용해야 immutable 컨셉에 맞다.

도커 허브가 아닌 따로 레지스트리를 사용한 경우 이미지 이름에 domain, port , name , version까지 다 포함해서 만들어야한다. 그래야 `docker push`할때 제대로 동작

## docker 실행

이미지를 만들엇으니 이제 실행해보자.

```bash
docker run --name my-test -d -p 80:80 test:0.1
```

-d : detach
-p : 호스트 포트번호 와 도커에서 expose한 포트번호를 콜론으로 구분해서 적어준다.

이제 nginx가 데몬으로 실행되는걸 알수 있다.

## 도커 안쪽에 내용을 호스트로 복사

```bash
docker cp hello-nginx:/etc/nginx/nginx.conf ./
```

## 도커 내용 보기

```bash
docker inspect my-test

docker inspect --help

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
```

## 도커 저장소

도커 저장소는 다음 블로그에서 다루겠다.

## docker network

컨테이너들끼리 통신을 하려면 docker network를 하나 만든후 모든 컨테이너를 이 네트워크에 포함시키면 docker 이름으로 서로 접속이 가능하다.

```bash
docker network create test-network
docker run --name db -d --network test-network mysql
docker run --name nginx -d --network test-network nginx
```

nginx와 db라는 이름으로 서로 통신이 가능하다.

## 호스트 볼륨 마운트

도커에 호스트 볼륨을 마운트해서 사용해보자.

```bash
docker run -d  \
-v /Users/ragon/Desktop/test/data:/data \
test:0.1
```

호스트 경로를 절대경로로 넣어주자. 상대경로도 되는지는 확인안해봣으니 이름만 적으면 `docker volume ls`에서 나오는 이름을 사용해야함.

## 추가 명령어

```bash
# 전체 도커 프로세스를 멈추고 remove하기
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)

# 전체 도커 이미지 지우기
docker image rm -f $(docker image ls -q)
```
