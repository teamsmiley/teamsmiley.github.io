---
layout: post
title: 'kubernetes cert-manager'
author: teamsmiley
date: 2020-10-05
tags: [cicd]
image: /files/covers/blog.jpg
category: { kubernetes }
---

# kubernetes cert-manager

연속된 글입니다.

1. <https://teamsmiley.github.io/2020/09/30/kubespray-01-vagrant/>
1. <https://teamsmiley.github.io/2020/10/01/kubespray-02-install-kube-local-internal-loadbalancer/>
1. <https://teamsmiley.github.io/2020/10/02/kubespray-03-kube-with-haproxy/>
1. <https://teamsmiley.github.io/2020/10/04/kubernetes-multi-cluster/>
1. <https://teamsmiley.github.io/2020/10/05/kubernetes-cert-manager/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-metallb-ingress-nginx/>
1. <https://teamsmiley.github.io/2020/10/06/kubernetes-helm/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-prometheus-grafana/>
1. <https://teamsmiley.github.io/2020/10/08/kubernetes-log/>
1. <https://teamsmiley.github.io/2020/10/10/kubernetes-backup-velero/>

## helm을 사용하자.

```bash
brew install helm
```

## cert-manager

### install

```bash
helm repo add jetstack https://charts.jetstack.io
kubectl config use-context c2
kubectl create namespace cert-manager
kubectl config set-context --current --namespace cert-manager
helm install jetstack/cert-manager --namespace cert-manager --generate-name --set installCRDs=true
```

```bash
NAME: cert-manager-1601943552
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

도메인 관리를 아마존에서 하므로 아마존으로 만들었습니다. lets encrypt를 이용하여 dns를 통해서 도메인 소유를 확인하므로 아래 단계가 필요합니다.

아마존에서는 미리 access key와 secret key등을 만들어두셔야합니다. 다른 사이트는 다른 방식으로 처리가 가능합니다.

https://cert-manager.io/docs/configuration/acme/dns01/ 여기서 부터 한번 보시기 바랍니다. 중요한것은 도메인 관리회사에 api에 접속이 될수가 있어야한다는 것입니다. 이것부터 하시고 나머지 단계로 하시면됩니다.

lets encrypt에 구글이나 도메인관리회사에 api에 접속하는법에 대한 내용들 위주로 검색해보시면 금방 찾으실수잇을겁니다.

### aws secret 만든다.

```bash
kubectl config set-context --current --namespace cert-manager
# kcn cert-manager
```

```bash
AWS_ACCESS_KEY_ID=XXXXX
AWS_SECRET_ACCESS_KEY=XXXXX
echo ${AWS_SECRET_ACCESS_KEY} > password.txt
kubectl create secret generic aws-route53-creds --from-file=password.txt -n cert-manager
rm -f password.txt
```

### dns issuer 설치

```
vi le-dns-issuer-aws.yaml
```

```yml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: le-dns-issuer-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: 'teamsmiley@gmail.com'
    privateKeySecretRef:
      name: le-dns-issuer-staging
    solvers:
      - dns01:
          route53:
            region: us-west-1
            accessKeyID: XXXXXX
            secretAccessKeySecretRef:
              name: aws-route53-creds #여기 이름 중요 나중에 쓴다.
              key: password.txt

---
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: le-dns-issuer-live
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: 'teamsmiley@gmail.com'
    privateKeySecretRef:
      name: le-dns-issuer-live
    solvers:
      - dns01:
          route53:
            region: us-west-1
            accessKeyID: AKIAXW6HU27EZ4ZGQEBN
            secretAccessKeySecretRef:
              name: aws-route53-creds
              key: password.txt
```

```bash
kubectl apply -f ./le-dns-issuer-aws.yaml

kubectl get ClusterIssuer

NAME                    READY   AGE
le-dns-issuer-live      True    2m2s
le-dns-issuer-staging   True    2m2s
```

### 실제로 발급해보자.

```
kubectl create namespace xgrid-staging
vi tls-staging.yml
```

```yml
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: tls-staging
  namespace: xgrid-staging
spec:
  secretName: aws-route53-creds
  issuerRef:
    name: le-dns-issuer-staging
    kind: ClusterIssuer
  dnsNames:
    - staging.xgridcolo.com
```

```bash
k apply -f tls-staging.yml
```

issuerRef가 위에서 만든 le-dns-issuer-staging 이다 스테이징이므로 테스트가 가능하다. 스테이징으로 테스트를 꼭 해서 괞찮으면 나중에 live로 바꿔서 다시 하면된다.

```bash
kubectl describe certificate tls-staging -n xgrid-staging
```

결과가 아래처럼 나오면 된다

```bash
Spec:
  Dns Names:
    *.staging.xgridcolo.com
    staging.xgridcolo.com
  Issuer Ref:
    Kind:       ClusterIssuer
    Name:       le-dns-issuer-staging
  Secret Name:  aws-route53-creds
Status:
  Conditions:
    Last Transition Time:  2020-10-05T15:06:55Z
    Message:               Certificate is up to date and has not expired
    Reason:                Ready
    Status:                True
    Type:                  Ready
```

추가로 확인하고 싶으면

```bash
kubectl get pod -n cert-manager # cert-manager pod확인
kubectl logs cert-manager-1602273512-6c8fff6d74-mvhdb -f -n cert-manager # cain이나 webhook가 아닌 main pod를 체크해야한다.
```
