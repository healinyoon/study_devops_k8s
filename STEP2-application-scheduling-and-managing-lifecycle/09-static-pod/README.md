# Static Pod

### 개요

사용자가 실행시키지 않고 static pod의 위치를 지정하여 kubelet에 의해 실행되는 Pod이다. Static Pod 경로의 YAML을 변경만 해도 자동으로 kubelet에 의해 반영된다.

### Static Pod 위치 지정 방법

#### 1. Static Pod 위치를 지정해주는 파일을 알아보자.

`Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)` 출력을 통해 Static Pod의 경로를 명시해주는 파일을 알 수 있다.

```
$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sat 2020-11-21 12:42:01 UTC; 1 months 17 days ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 1114 (kubelet)
    Tasks: 19 (limit: 9479)
   CGroup: /system.slice/kubelet.service
           └─1114 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2

```

#### 2. `/lib/systemd/system/kubelet.service` 파일 수정

해당 파일을 수정하면 Static Pod 경로를 추가할 수 있다.

```
# vi /lib/systemd/system/kubelet.servic

ExecStart= 에 --pod-manifest-path=/etc/kubelet.d 추가
```

#### 3. daemon reload 및 kubelete restart

```
# systemctl daemon-reload
# systemctl restart kubelet
```

# Static Pod의 기본 경로

* `/etc/kubernetes/manifests`
* Kubernetes system의 주요 기능을 위해 이미 다음과 같은 Static Pod를 사용하고 있다.

```
$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

* 각 component의 세부 사항을 설정할 때는 위의 경로의 파일을 수정하면 자동으로 업데이트되어 Pod를 재구성한다.
* Pod 작성 요령은 기존의 Pod 작성법과 동일하다.