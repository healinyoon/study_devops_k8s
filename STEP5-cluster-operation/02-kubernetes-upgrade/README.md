# Kubernetes 버전 업그레이드

### Kubernetes 버전이 나타내는 내용
* Major
* Minor: 특징, 기능
* Patch: 버그 픽스

![](/STEP5-cluster-operation/images/02-kubernetes-upgrade-v.png)

### 모든 버그를 고치기 위해 alpha와 beta로 나누어 출시 후 정식 버전 진행
* alpha: 추가되는 기능들을 disable한 형태로 배포
* beta: 추가되는 기능들을 enable한 형태로 배포
* release: 안정화된 버전을 배포

### 패키지마다 모든 Component는 동일한 버전으로 배포
* API Server, Controller Manager, Scheduler, Kubelet, Kube Proxy 등
    * 즉 Master Server를 업데이트 하면 위의 컴포넌트도 자동으로 업데이트 된다.
* ETCD, 네트워크, CoreDNS는 제외 
    * 외부 Project이기 때문이다.

### Kubernetes 버전 호환
Kubernetes는 apiserver를 기준으로 호환성을 제공한다. 무중단 운영을 위해서는 각각의 Node를 Master Node부터 Worker Node까지 하나씩 업그레이드 해야 한다(모든 Node 한번에 NO!!!). 따라서 호환되는 버전을 확인해야 한다.

| Application | 지원 버전 범위(apiserver minor version x) | 예시(apiserver version 1.13로 업그레이드 하려는 경우) |
| --- | --- | --- |
| kube-apiserver | x, x-1 | 1.13, 1.12 |
| kubelet | x, x-1, x-2 | 1.13, 1.12, 1.11 |
| kube-controller-manager<br/>kube-scheduler<br/>cloud-controller-manager | x, x-1 | 1.13, 1.12 |
| kubectl | x+1, x, x-1 | 1.14, 1.13, 1.12 |

**x-1**까지는 모두 공통적으로 호환을 지원해주는 것을 알 수 있다.


### Kubernetes 버전 업그레이드 방법
Node를 하나씩 Drain하며 버전을 업데이트한다(RollingUpdate). 업데이트를 적용할 때는 minor 버전이 하니씩 업데이트 되도록 설정하는 것이 좋다.

```
master $ kubectl drain node-1
master $ ssh node-1

node-1 $ apt-get upgrade -y kubeadm=1.12.0-00
node-1 $ apt-get upgrade -y kubelet=1.12.0-00

// kubeadm node 업그레이드: 내부 api-server를 비롯한 container로 존재하는 image들을 업그레이드 시켜준다
node-1 $ kubeadm upgrade node config --kubelet-version v1.12.0 
node-1 $ systemctl restart kubelet
node-1 $ exit

master $ kubectl uncordon node-1
```

# Kubernetes 버전 업그레이드 실습

[kubeadm 클러스터 업그레이드 공식 문서](https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

### Master Node 버전 업그레이드

Master부터 하나씩 버전 업그레이드를 진행한다.

#### 1. Node 확인
```
$ kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   128d   v1.19.0
worker1   Ready    <none>   128d   v1.19.0
worker2   Ready    <none>   128d   v1.19.0
```

#### 2. Master Node Drain
```
$ kubectl drain master --delete-local-data --ignore-daemonsets
node/master already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-49zfz, kube-system/weave-net-q6vj7
evicting pod default/http-go-568f649bb-wzzm4
evicting pod default/http-go-568f649bb-n7w9l
evicting pod default/http-go-568f649bb-vj6bn
evicting pod default/http-go-568f649bb-wpj2s
evicting pod default/ratings-v1-7d99676f7f-jsbs7
evicting pod default/http-go-568f649bb-xcdmr
evicting pod default/productpage-v1-65576bb7bf-l7csc
evicting pod istio-system/kiali-7476977cf9-rwmdw
evicting pod default/reviews-v2-6c5bf657cf-llzfj
evicting pod kube-system/coredns-f9fd979d6-g8t5z
evicting pod kube-system/metrics-server-75f98fdbd5-4kbmw
I0114 09:05:21.204748   44621 request.go:645] Throttling request took 1.043001516s, request: GET:https://10.1.11.7:6443/api/v1/namespaces/default/pods/http-go-568f649bb-wpj2s
I0114 09:05:31.404704   44621 request.go:645] Throttling request took 1.362004025s, request: GET:https://10.1.11.7:6443/api/v1/namespaces/default/pods/http-go-568f649bb-vj6bn
pod/http-go-568f649bb-vj6bn evicted
pod/http-go-568f649bb-n7w9l evicted
pod/http-go-568f649bb-wpj2s evicted
pod/http-go-568f649bb-xcdmr evicted
pod/coredns-f9fd979d6-g8t5z evicted
pod/metrics-server-75f98fdbd5-4kbmw evicted
pod/kiali-7476977cf9-rwmdw evicted
pod/productpage-v1-65576bb7bf-l7csc evicted
pod/reviews-v2-6c5bf657cf-llzfj evicted
pod/http-go-568f649bb-wzzm4 evicted
pod/ratings-v1-7d99676f7f-jsbs7 evicted
node/master evicted
```

#### 3. kubeadm 버전 업그레이드
```
$ sudo apt-mark unhold kubeadm kubelet && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.20.0-00 && sudo apt-get install -y kubelet=1.20.0-00 && \
sudo apt-mark hold kubeadm kubelet
```

#### 4. 버전 확인
```
$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.0", GitCommit:"af46c47ce925f4c4ad5cc8d1fca46c7b77d13b38", GitTreeState:"clean", BuildDate:"2020-12-08T17:57:36Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

#### 5. 업그레이드 계획 확인
```
$ sudo kubeadm upgrade plan
```

#### 6. kubeadm 업그레이드 호출
```
$ sudo kubeadm upgrade apply v1.20.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.20.0"
[upgrade/versions] Cluster version: v1.19.0
[upgrade/versions] kubeadm version: v1.20.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]:  y
```

마지막으로 아래 메세지가 출력된다.
```
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.20.0". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

**주의: 다른 Master Node의 경우**

두 번째 Master Node부터는 위의 명령어 대신 아래의 명령어를 사용한다.

```
sudo kubeadm upgrade node
```

#### 7. kubelet restart
```
$ sudo systemctl restart kubelet
```

#### 8. Node 확인
```
$ kubectl get nodes
NAME      STATUS                        ROLES                  AGE    VERSION
master    NotReady,SchedulingDisabled   control-plane,master   128d   v1.20.2
worker1   Ready                         <none>                 128d   v1.19.0
worker2   Ready                         <none>                 128d   v1.19.0
```

#### 9. Master Node Uncordon
```
$ kubectl uncordon master
node/master uncordoned
```

#### 10. Node 확인
```
$ kubectl get nodes
NAME      STATUS   ROLES                  AGE    VERSION
master    Ready    control-plane,master   128d   v1.20.2
worker1   Ready    <none>                 128d   v1.19.0
worker2   Ready    <none>                 128d   v1.19.0
```

버전 업그레이드가 잘 된 것을 확인할 수 있다.


### Worker Node 버전 업그레이드

#### 1. Worker Node Drain
```
$ kubectl drain worker1 --ignore-daemonsets
```

#### 2. kubeadm 업그레이드
```
$ sudo apt-mark unhold kubeadm kubelet && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.20.0-00 && sudo apt-get install -y kubelet=1.20.0-00 && \
sudo apt-mark hold kubeadm kubelet
```

#### 3. kubeadm 업그레이드 호출
```
$ sudo kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```

#### 4. kubelet restart
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```

#### 5. Node 확인
```
$ kubectl get nodes
NAME      STATUS                     ROLES                  AGE    VERSION
master    Ready                      control-plane,master   128d   v1.20.2
worker1   Ready,SchedulingDisabled   <none>                 128d   v1.19.0
worker2   Ready                      <none>                 128d   v1.19.0
```

#### 6. Worker Node Uncordon
```
$ kubectl uncordon worker1
node/worker1 uncordoned
```

#### 7. Node 확인
```
$ kubectl get nodes
NAME      STATUS   ROLES                  AGE    VERSION
master    Ready    control-plane,master   128d   v1.20.2
worker1   Ready    <none>                 128d   v1.20.0
worker2   Ready    <none>                 128d   v1.19.0
```

버전 업그레이드가 잘 된 것을 확인할 수 있다.