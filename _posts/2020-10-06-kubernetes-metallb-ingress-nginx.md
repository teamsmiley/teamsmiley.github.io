---
layout: post
title: "kubernetes MetalLB 와 Ingress Nginx"
author: teamsmiley
date: 2020-10-06
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# kubernetes MetalLB 와 Ingress nginx

## metallb (베어메탈에서 사용하는 로드발란서)

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

kubectl edit configmap -n kube-system kube-proxy

```yml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true #false를 true로 변경
```

또는 자동화

```bash
# 보기
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# 변경
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

이제 설치

Installation By Manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

혹시 버전이 바뀔지 모르니 백업을 해두자.

```bash
cd ~/Desktop/kubespray/inventory/metallb
curl -O https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
curl -O https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
kubectl apply -f default.yaml
kubectl apply -f metallb-install.yaml
```

이제 설정

vi metallb-configmap.yml

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.2.11-192.168.2.99
```

```bash
kubectlapply -f metallb-configmap.yml
```

## Ingress-Nginx

helm repo list
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list

kubectl config set-context --current --namespace= ingress-nginx

helm install ingress-nginx ingress-nginx/ingress-nginx

아래 내용이 나온다 잘 읽어보자.

```yml
NAME: ingress-nginx
LAST DEPLOYED: Tue Oct  6 17:12:45 2020
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: example
  namespace: foo
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - backend:
              serviceName: exampleService
              servicePort: 80
            path: /
  # This section is only required if TLS is to be enabled for the Ingress
  tls:
      - hosts:
          - www.example.com
        secretName: example-tls
---
If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

이 내용처럼 ingress를 나중에 만들면된다.

확인하려면 다음 실행

```bash
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
```

현재 metallb에 첫번째 아이피 11번을 받아온걸 알수 있다.

그런데 아이피가 바뀌면 안되는것이라..

서비스 타입은 loadbalancer 그리고 loadBalancerIP를 지정해야할듯 하다.

kubectldelete svc ingress-nginx-controller

ingress-service.yml

```yml
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: ingress-nginx
    meta.helm.sh/release-namespace: ingress-nginx
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/version: 0.40.2
    helm.sh/chart: ingress-nginx-3.4.1
  name: ingress-nginx-controller
  namespace: ingress-nginx
  resourceVersion: "41715"
  selfLink: /api/v1/namespaces/ingress-nginx/services/ingress-nginx-controller
  uid: a6d85078-4318-42ec-aa57-cca529a0c1b6
spec:
  loadBalancerIP: 192.168.2.99 #metallb에서 가능한 아이피중 하나
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
```

```
kubectlapply -f ingress-service.yml
```

vi sample-ingress.yml

```yml
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: example
  namespace: default
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - backend:
              serviceName: example-svc
              servicePort: 80
            path: /
```

```
kubectlapply -f sample-ingress.yml
```

Error

```
Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io"
```

해결책

```
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

다시 적용 잘된다.

```
kubectlapply -f sample-ingress.yml
```

www.example.com을 Hosts파일에 저장하고 192.168.2.99를 적용한다.

www.example.com을 웹브라우저에 요청을 하면 웹 페이지가 뜨면서 에러가 난다.

이건 맞는것이다 ingress는 동작하고 그것이 실제 서비스까지 연결이 안되서 생기는 문제이다.

kubectlget ingress로 확인이 가능하다.
