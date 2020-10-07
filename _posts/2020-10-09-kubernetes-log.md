---
layout: post
title: "kubernetes - 로그 분석 elasticserach kibana fluentbit"
author: teamsmiley
date: 2020-10-08
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>
1. <https://teamsmiley.github.io/2020/10/05/kubernetes-cert-manager/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-metallb-ingress-nginx/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-helm/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-prometheus-grafana/>

# kubernetes - Log를 분석해보자.

1. fluentd가 로그를 수집해서 elasticsearch에 보내면
1. elastic search는 로그를 저장하고
1. kibana가 elastic search를 디비로 해서 화면에 로그를 뿌려줌

## create namespace

```bash
k create namespace logging
kcn logging
```

꼭 logging을쓰기 바란다. 설치파일중에 namespace 가 고정된 부분이 있다.

## elastic search

### install

```bash
helm repo add elastic https://helm.elastic.co
```

일단 persistence가 없는걸로 해서 진행해본다.

```bash
helm install elasticsearch elastic/elasticsearch \
--set persistence.enabled=false --namespace logging

kubectl get pods --namespace=monitoring -l app=elasticsearch-master -w #상태 모니터링

kubectl describe svc elasticsearch-master
```

확인해보자.
포트 포워딩

```
kubectl --namespace logging port-forward svc/elasticsearch-master 9200
```

<http://localhost:9200/>

![]({{ site_baseurl }}/assets//2020-10-07-08-57-16.png)

동작한다.

## kibana

### install

```bash
helm install kibana elastic/kibana --namespace logging
```

확인해보자.
포트 포워딩

```bash
kubectl --namespace logging port-forward svc/kibana-kibana 5601
```

<http://localhost:5601/app/home>

![]({{ site_baseurl }}/assets//2020-10-07-09-06-30.png)

잘 보인다.

## Fluent Bit

Fluentd is a log collector, processor, and aggregator.
Fluent Bit is a log collector and processor (it doesn’t have strong aggregation features such as Fluentd).

Fluentd를 경량화해서 메모리도 적게 쓰고 cpu도 적게쓰는 버전이 fluent bit이다.

<https://github.com/fluent/fluent-bit-kubernetes-logging/>

### install

daemonset으로 올려서 모든 노드에 설치가 되게 한다.

```bash
kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-service-account.yaml
kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role.yaml
kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/fluent-bit-role-binding.yaml
```

configmap을 만든다.

```
kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-configmap.yaml
```

이제 elastic search으로 보내자.

```
kubectl create -f https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds.yaml
```

에러가 혹시 나면 configmap에서 꼭 elastic search ip port를 줘야한다.

```
[OUTPUT]
    Name            es
    Match           *
    Host            elasticsearch-master.logging.svc.cluster.local
    Port            9200
    Logstash_Format On
    Replace_Dots    On
    Retry_Limit     False
```

## kibana로 로그가 들어오는지 확인하자.

localhost:5601 에서 kibana >> index pattern >> logstash-\* 추가하자.

![]({{ site_baseurl }}/assets//2020-10-07-14-52-54.png)

![]({{ site_baseurl }}/assets//2020-10-07-14-53-14.png)

잠시후

![]({{ site_baseurl }}/assets//2020-10-07-15-36-48.png)

![]({{ site_baseurl }}/assets//2020-10-07-15-37-08.png)

결과가 잘 나온다. 사용법은 다음에