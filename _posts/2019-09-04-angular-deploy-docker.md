---
layout: post
title: 'angular deploy with docker (nginx)' 
author: teamsmiley
date: 2019-09-04
tags: [devops]
image: /files/covers/blog.jpg
category: {business}
---
# angular deploy with docker (nginx)

문제점 : angular를 도커로 패키징을 해서 사용중인데 f5를 할때마다 404에러가 난다. 

원인 : 쿠버네티스가 aaa.com/suburl 을 넘기는데 nginx가 디렉토리로 인식하고 404를 보낸다. 

해결 방법 : 

도커를 사용해서 deploy를 하려고 한다. 

```
# stage 1
FROM node:12 as builder

# use changes to package.json to force Docker not to use the cache
# when we change our application's nodejs dependencies:
COPY package.json /tmp/package.json
RUN cd /tmp && npm install
RUN mkdir -p /app && cp -a /tmp/node_modules /app

WORKDIR /app
COPY . .
# RUN npm install
RUN npm run build:development

FROM nginx:alpine
COPY --from=builder /app/dist/ /usr/share/nginx/html/

COPY ./nginx.conf /etc/nginx/conf.d/default.conf //중요

EXPOSE 80
```

nginx.conf를 만든다.
```
server {
  listen 80;
  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html =404;
  }
}
```

try_files 가 중요하다. 

이제 도커 빌드를 해서 레지스트리에 올리면된다.

```bash
IMAGE_NAME=registry.aaa.com:5000/www-dev:$CI_JOB_ID
docker build -f ./Dockerfile-dev -t $IMAGE_NAME .
docker push $DEV_IMAGE_NAME
```

완료

이제 f5를 해도 404가 나오면 일단 index.html을 시도해서 결과 나오면 보내준다. 

잘된다.





