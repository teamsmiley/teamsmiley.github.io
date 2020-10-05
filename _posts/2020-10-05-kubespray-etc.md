---
layout: post
title: "kubespray - 05 추가 작업"
author: teamsmiley
date: 2020-10-05
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# kubespray - 05 추가 작업

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>

이제 추가 작업에 대해서 적어보자.

## helm을 사용하자.

```
brew install helm
```

## cert-manager 설치

```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.2/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
kubectl create namespace cert-manager
helm install my-release --namespace cert-manager jetstack/cert-manager
```

```bash
NAME: my-release
LAST DEPLOYED: Sun Oct  4 22:05:04 2020
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

## ingress-nginx

## prometheus flentd grafana

##
