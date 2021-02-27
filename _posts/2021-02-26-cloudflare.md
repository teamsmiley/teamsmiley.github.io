---
layout: post
title: "cloudflare api"
author: teamsmiley
date: 2021-02-26
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# CloudFlare api 사용하기

CloudFlare에서 Spectrum 서비스를 이용하기위해 port를 등록 해야하는데 200여개 포트를 하나씩 등록하면 너무 힘들어서 api를 사용하기로 했다.

로그인후 dashboard에서 api key를 발급 받는다.

https://dash.cloudflare.com/profile/api-tokens

api keys >> global api key 발급

실제 api에 대한 자세한 설명은 다음 링클르 보면 된다.

<https://api.cloudflare.com/#getting-started-endpoints>

## zone id

먼저 zone id를 알고 있어야한다. 확인하기 위해서

```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones" \
 -H "Content-Type:application/json" \
 -H "X-Auth-Key:YOUR_API_KEY" \
 -H "X-Auth-Email:YOUR_EMAIL"
```

결과는 다음과 같다.

```json
{
  "result":[
    {
      "id":"xxxa0ad5ca54c0c2be85e6ff019132a1",
      "name":"aaa.com",
      "status":"active",
      "paused":false,
      "type":"full"
```

## spectrum(클라우드플레어 서비스중 하나) : tcp proxy

관련 api는 다음과 같다.

<https://api.cloudflare.com/#spectrum-applications-properties>

- 리스트를 가져와보자.

GET zones/:zone/spectrum/apps

```
curl -X GET "https://api.cloudflare.com/client/v4/zones/xxx0ad5ca54c0c2be85e6ff019132a1/spectrum/apps" \
 -H "Content-Type:application/json" \
 -H "X-Auth-Key:xxxc0bb71dabf169459d36f8d0ba5e889b798" \
 -H "X-Auth-Email:YOUR_EMAIL"
```

포트를 등록한 리스트가 쭉 나온다.

- 이제 생성해보자.

POST zones/:zone/spectrum/apps

```
curl -X POST "https://api.cloudflare.com/client/v4/zones/xxx0ad5ca54c0c2be85e6ff019132a1/spectrum/apps" \
  -H "Content-Type:application/json" \
  -H "X-Auth-Key:xxxc0bb71dabf169459d36f8d0ba5e889b798" \
  -H "X-Auth-Email:YOUR_EMAIL"
```

대충 이런식으로 해서 아래 데이터를 보내주면 된다.

나는 postman으로 사용을 해서 테스트했더니 잘 됨.

```json
{
  "ip_firewall": false,
  "proxy_protocol": "off",
  "tls": "off",
  "traffic_type": "direct",
  "edge_ips": {
    "type": "dynamic",
    "connectivity": "ipv4"
  },
  "dns": {
    "type": "CNAME",
    "name": "test.yourdomain.com"
  },
  "protocol": "tcp/63111",
  "origin_direct": ["tcp://28.16.0.xxx:63111"]
}
```
