---
layout: post
title: "kubernete cgroup-driver"
author: teamsmiley
date: 2020-09-27
tags: [cicd]
image: /files/covers/blog.jpg
category: { kube }
---

# kubernete cgroup-driver Issue

kubernetes를 설치하면 잘 되는데 재부팅시 not ready가 되었다.

에러를 찾아봤다.

```bash
systemctl status kubelet

systemctl restart kubelet

systemctl status kubelet
```

```bash
kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since Tue 2020-09-29 05:49:32 PDT; 8s ago
     Docs: https://kubernetes.io/docs/
  Process: 16256 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 16256 (code=exited, status=255)
```

정확한 에러는 안나와서 `journalctl -xefu kubelet` 실행

```bash
failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
```

아하..이런 에러구나.

kubernetes설치시 docker cgroup driver를 systemd로 하라고 가이드한다.

그래서 다음처럼 해뒀었다.

```bash
cat > /etc/docker/daemon.json <{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

이제 도커는 systemd를 cgroupdriver로 사용한다.

정작 kube는 설정이 안됬나보다. 이상한게 재부팅하면 문제가 발생한다는것이다.

vi /var/lib/kubelet/kubeadm-flags.env

```bash
#KUBELET_KUBEADM_ARGS="--cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1"
KUBELET_KUBEADM_ARGS="--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1"
```

이제 재시작해보자.

```bash
systemctl restart kubelet

systemctl status kubelet

● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2020-09-29 06:11:27 PDT; 4s ago
```

잘 된다.
