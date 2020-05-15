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
---
apiVersion: v1
kind: Namespace
metadata:
  name: elastic-search
```

* elastic-search.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: elastic-search
  labels:
    app: elasticsearch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: elastic/elasticsearch:7.7.0
        env:
        - name: discovery.type
          value: "single-node"
        ports:
        - containerPort: 9200
        - containerPort: 9300
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch-svc
  namespace: elastic-search
spec:
  ports:
  - name: elasticsearch-rest
    nodePort: 30482
    port: 9200
    protocol: TCP
    targetPort: 9200
  - name: elasticsearch-nodecom
    nodePort: 30930
    port: 9300
    protocol: TCP
    targetPort: 9300
  selector:
    app: elasticsearch
  type: NodePort
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
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  labels:
    app: kibana
  namespace: elastic-search
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: elastic/kibana:7.7.0
        env:
        - name: SERVER_NAME
          value: "kibana.kubenetes.example.com"
        - name: ELASTICSEARCH_HOSTS
          value: "http://elasticsearch-svc.elastic-search.svc.cluster.local:9200"
        ports:
        - containerPort: 5601
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kibana
  name: kibana-svc
  namespace: elastic-search
spec:
  ports:
  - nodePort: 30920
    port: 5601
    protocol: TCP
    targetPort: 5601
  selector:
    app: kibana
  type: NodePort
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
*fluentd.yml
```yml

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: fluentd
  name: fluentd
  namespace: kube-system

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
rules:
  - apiGroups:
      - ""
    resources:
      - "namespaces"
      - "pods"
    verbs:
      - "list"
      - "get"
      - "watch"

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: fluentd
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.4.2-debian-elasticsearch-1.1
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch-svc.elastic-search.svc.cluster.local"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "http"
          - name: FLUENTD_SYSTEMD_CONF
            value: "disable"
          - name: FLUENT_UID
            value: "0"
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

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



