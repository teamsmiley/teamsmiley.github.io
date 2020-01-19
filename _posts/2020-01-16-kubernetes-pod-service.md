---
layout: post
title: 'kubernetes pod service에서 연결하기' 
author: teamsmiley
date: 2020-01-16
tags: [kubernetes]
image: /files/covers/blog.jpg
category: {kuberentes}
---

mysql을 쿠버네티스에 배포 연습

Run a Replicated Stateful Application

<https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/>

pv가 dynamic provision이 되면 아주 잘 된다.

그런데 마지막에 문제가 master 에 접속을 하는것인데
```
For writes, you must instead connect to the master: mysql-0.mysql.
```

쿠버네티스 내부에서는 이렇게 mysql-0.mysql로 접속하면 된다. 그럼 dns lookup을 통해 잘 된다. 

문제는 외부에서 접속할때 어디로 가야할지 잘 모르겟다는것이다. 

지인에게 물어본결과 다음처럼 하면된다.

먼저 pod에 label을 가져온다.
```
kubectl get pods --show-labels 

NAME                  READY   STATUS    RESTARTS   AGE    LABELS
mysql-0               2/2     Running   0          8h     app=mysql,controller-revision-hash=mysql-8668bd9989,statefulset.kubernetes.io/pod-name=mysql-0
mysql-1               2/2     Running   0          8h     app=mysql,controller-revision-hash=mysql-8668bd9989,statefulset.kubernetes.io/pod-name=mysql-1
mysql-2               2/2     Running   0          8h     app=mysql,controller-revision-hash=mysql-8668bd9989,statefulset.kubernetes.io/pod-name=mysql-2
```

mysql-0 에서 label이름을 보면 statefulset.kubernetes.io/pod-name=mysql-0 이렇게 나온다. 이걸 이제 외부에 오픈하는 서비스에서 붙여주면된다.

service.yml
```yml
apiVersion: v1
kind: Service
metadata:
  name: external-mysql-0
  namespace: kube-system
  labels:
    app: mysql
spec:
  type: NodePort
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 33306
  selector:
    statefulset.kubernetes.io/pod-name: mysql-0
```

selector 를 사용해서 서비스에서 붙여준다.




