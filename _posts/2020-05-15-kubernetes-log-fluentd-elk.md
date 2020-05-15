---
layout: post
title: 'kubernetes 로그를 fluentd로 보내고 elasticsearch에 저장하고 kibana로 확인' 
author: teamsmiley
date: 2020-05-14
tags: [Kubernetes]
image: /files/covers/blog.jpg
category: {kubernetes}
---

# kubernetes 로그를 fluentd로 보내고 elasticsearch에 저장하고 kibana로 확인

전체 과정은 아래와 같다. 

kubernetes node 로그 -> fluentd -> elasticsearch -> kibana

## elasticsearch 
* default.yml
```yml

```

* elastic-search.yml
```yml

```

```bash
k apply -f default.yml
k apply -f elastic-search.yml
k get pod -o wide

NAME                             READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
elasticsearch-5fcd8ff46f-224cv   1/1     Running   0          9s    10.47.128.9   node10   <none>           <none>
```

확인 <http://192.168.0.10:30482/>



## kibana
* kibina.yml
```yml

```

```bash
k apply -f kibana.yml
k get pod 
```

연결하는 elasticsearch 주소는 http://elasticsearch-svc.elastic-search.svc.cluster.local:9200 이게 중요

error 

PollError No Living connections

ELASTICSEARCH_URL 이 잘못된경우. 다음처럼 확인하자.
```
http://elasticsearch-svc.elastic-search.svc.cluster.local:9200 
elasticsearch-svc : svc name
elastic-search    : namespace
```

확인 <http://192.168.0.10:30920/>

잘된다.

## fluentd를 사용하여 로그 업로드하자.

전체 노드에 daemonset으로 설치한다.

```bash
k apply -f fluentd.yml

serviceaccount/fluentd created
clusterrole.rbac.authorization.k8s.io/fluentd created
clusterrolebinding.rbac.authorization.k8s.io/fluentd created
daemonset.apps/fluentd created
```

/var/log를 모두 로그를  http://elasticsearch-svc.elastic-search.svc.cluster.local:9200에 저장 한다.

```bash
k get daemonset --all-namespaces

NAMESPACE        NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system      fluentd         14        14        14      14           14          <none>                        67s
```

14대에 모두 설치 완료

로그 확인. 노드 한대에 들어가서 

```bash
k logs -f 440fa8c333eb

[warn]: #0 [out_es] Could not communicate to Elasticsearch, resetting connection and trying again. getaddrinfo: Name or service not known (SocketError)
```

elastic-search주소 잘 확인 
```
k get pod -o wide --all-namespaces | grep fluentd
```
잘 됨. 


## kibana 설정

```
Management -> kibana -> Index Patterns -> Create Index Pattern -> logstash-* -> next step -> @timestamp
```

데이터 확인

메뉴에서 discover 클릭

잘 나온다..

이제 뭐하지?


## todo 
* kubernetes로그는 모아서 볼수 있으나 어플리케이션 로그들은 어떻게 모으지?
* 메모리 부족 문제
* 디스크 부족 문제
* 에러만 볼수 있나? 워닝만 볼수 있나?
* X-Pack is an Elastic Stack extension 활성화



