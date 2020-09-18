# Application log 관리

### Kuberentes application log 확인

* Log는 Container 단위로 확인 가능하다.
* Single container Pod의 경우 Pod까지만 지정하여 log 확인
* Multi container Pod의 경우 Container 이름을 옵션으로 전달하여 log 확인
  * `kubectl logs {Pod 명} {옵션: Container 명}`

### Kubeapi가 정상 동작하지 않는 경우 => kubectl 명령어 사용 불가

Kubernetes에서 동작하는 모든 리소스는 docker 사용한다. 따라서 kubectl 명령어를 사용할 수 없을 땐 docker의 로깅 기능을 사용하면 된다. 

* Container 조회
```
# docker ps -a
```

* Container log 조회
```
# docker logs {Container ID}
```

# Kubelet이 log를 남기는 동작 확인해보기

### Kubelete 동작 확인
```
$ journalctl -u kubelet
-- Logs begin at Tue 2020-09-01 13:35:28 UTC, end at Fri 2020-09-18 04:37:52 UTC. --
Sep 02 07:24:48 k8s-master01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
Sep 02 07:24:48 k8s-master01 systemd[1]: kubelet.service: Current command vanished from the unit file, execution of the command list won't be resumed.
Sep 02 07:24:48 k8s-master01 systemd[1]: Stopping kubelet: The Kubernetes Node Agent...
Sep 02 07:24:48 k8s-master01 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
Sep 02 07:24:48 k8s-master01 systemd[1]: Started kubelet: The Kubernetes Node Agent.
Sep 02 07:24:48 k8s-master01 kubelet[38168]: F0902 07:24:48.677317   38168 server.go:198] failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/
Sep 02 07:24:48 k8s-master01 kubelet[38168]: goroutine 1 [running]:
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/vendor/k8s.io/klog/v2.stacks(0xc0000c4001, 0xc0007fc000, 0xfb, 0x14d)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/klog/v2/klog.go:996 +0xb9
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/vendor/k8s.io/klog/v2.(*loggingT).output(0x6cf6140, 0xc000000003, 0x0, 0x0, 0xc0007c8a10, 0x6b4854c, 0x9, 0xc6, 0xc0006a2200)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/klog/v2/klog.go:945 +0x191
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/vendor/k8s.io/klog/v2.(*loggingT).printDepth(0x6cf6140, 0x3, 0x0, 0x0, 0x1, 0xc000abfd00, 0x1, 0x1)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/klog/v2/klog.go:718 +0x165
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/vendor/k8s.io/klog/v2.(*loggingT).print(...)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/klog/v2/klog.go:703
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/vendor/k8s.io/klog/v2.Fatal(...)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/k8s.io/klog/v2/klog.go:1436
Sep 02 07:24:48 k8s-master01 kubelet[38168]: k8s.io/kubernetes/cmd/kubelet/app.NewKubeletCommand.func1(0xc000995340, 0xc0000c6010, 0x3, 0x3)
Sep 02 07:24:48 k8s-master01 kubelet[38168]:         /workspace/anago-v1.19.0-rc.4.197+594f888e19d8da/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubelet/app/server.go:198 +0xa05
```

### docker로 돌아가고 있는 Kuberentes 프로세스 확인하기
```
$ pstree
systemd─┬─accounts-daemon───2*[{accounts-daemon}]
        ├─2*[agetty]
        ├─atd
        ├─containerd─┬─containerd-shim─┬─pause
        │            │                 └─10*[{containerd-shim}]
        │            ├─5*[containerd-shim─┬─pause]
        │            │                    └─9*[{containerd-shim}]]
        │            ├─containerd-shim─┬─etcd───13*[{etcd}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─kube-proxy───6*[{kube-proxy}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─kube-utils───7*[{kube-utils}]
        │            │                 ├─launch.sh───weaver───14*[{weaver}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─weave-npc─┬─ulogd
        │            │                 │           └─8*[{weave-npc}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─kube-controller───6*[{kube-controller}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─kube-scheduler───8*[{kube-scheduler}]
        │            │                 └─9*[{containerd-shim}]
        │            ├─containerd-shim─┬─kube-apiserver───9*[{kube-apiserver}]
        │            │                 └─9*[{containerd-shim}]
        │            └─23*[{containerd}]
(중략)
```

=> 그래서 결론은 log가 docker의 경로에 쌓인다는 말이다.

# Docker log 쌓여있는 것 살펴보기

#### 경로로 이동
```
$ cd /var/log/containers/
```

#### 목록 확인
```
$ ll
total 44
drwxr-xr-x  2 root root   4096 Sep 17 01:22 ./
drwxrwxr-x 13 root syslog 4096 Sep 17 06:25 ../
lrwxrwxrwx  1 root root     81 Sep  8 07:35 etcd-master_kube-system_etcd-6a0f640e457e17f0e3589486efaaab388113b1679a249a3c6dfb89626f4ab95c.log -> /var/log/pods/kube-system_etcd-master_5f1924a86339fb52ac1fc0ed5c10b18e/etcd/0.log
lrwxrwxrwx  1 root root    101 Sep 17 01:22 kube-apiserver-master_kube-system_kube-apiserver-c8b1d142a7c150594ed4687b9a8f1fd7848555942eb51bf4451b2a25976e2877.log -> /var/log/pods/kube-system_kube-apiserver-master_6630e1d80adb2b1bbbe817cad3d93d7c/kube-apiserver/0.log
lrwxrwxrwx  1 root root    119 Sep 17 01:22 kube-controller-manager-master_kube-system_kube-controller-manager-00974f13657b7f4a9d4a9c02a1a25f2875159ade9990875e32f39ff16edbcdb4.log -> /var/log/pods/kube-system_kube-controller-manager-master_6364e43ef3555c356a1732732a00b65f/kube-controller-manager/3.log
lrwxrwxrwx  1 root root    119 Sep 11 05:52 kube-controller-manager-master_kube-system_kube-controller-manager-03eb1dbd4aeca87837ab68b3b2334619b4c40948694e4cd384ad375c0359631b.log -> /var/log/pods/kube-system_kube-controller-manager-master_6364e43ef3555c356a1732732a00b65f/kube-controller-manager/2.log
lrwxrwxrwx  1 root root     96 Sep  8 07:35 kube-proxy-49zfz_kube-system_kube-proxy-dcaf6a1fb45ec078a055d5976d6701b155d2a94c03d2e21b2881c5d9863f5dcb.log -> /var/log/pods/kube-system_kube-proxy-49zfz_b31b76d8-db32-44f6-9d78-6b81c9ee5391/kube-proxy/0.log
lrwxrwxrwx  1 root root    101 Sep 11 05:52 kube-scheduler-master_kube-system_kube-scheduler-45ae4cb5fb769294ddd6a1acb3fd56fded25ee4992d2114e057754fb658927a0.log -> /var/log/pods/kube-system_kube-scheduler-master_5146743ebb284c11f03dc85146799d8b/kube-scheduler/2.log
lrwxrwxrwx  1 root root    101 Sep 17 01:22 kube-scheduler-master_kube-system_kube-scheduler-b519d08360e4fa8ee17c86a20b459247ee5ab8b58c6f359c4a1971b9d073e5f6.log -> /var/log/pods/kube-system_kube-scheduler-master_5146743ebb284c11f03dc85146799d8b/kube-scheduler/3.log
lrwxrwxrwx  1 root root     90 Sep  8 07:46 weave-net-q6vj7_kube-system_weave-8afa0246e6cfdbae1c19f19a1331083c19794c767a6401f55ecad0a302fcadc0.log -> /var/log/pods/kube-system_weave-net-q6vj7_fddb8017-e841-46dc-9642-7b20a99f7078/weave/0.log
lrwxrwxrwx  1 root root     94 Sep  8 07:46 weave-net-q6vj7_kube-system_weave-npc-07970c3cdb162d428ba13f7be372847284b8e567f16b3eaeb05e59a4fc62d0bc.log -> /var/log/pods/kube-system_weave-net-q6vj7_fddb8017-e841-46dc-9642-7b20a99f7078/weave-npc/0.log
```

#### Log 내용 보기
```
$ cat weave-net-q6vj7_kube-system_weave-8afa0246e6cfdbae1c19f19a1331083c19794c767a6401f55ecad0a302fcadc0.log
{"log":"INFO: 2020/09/08 07:46:28.658904 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true host-root:/host http-addr:127.0.0.1:6784 ipalloc-init:consensus=2 ipalloc-range:10.32.0.0/12 metrics-addr:0.0.0.0:6782 name:de:60:e9:1e:f8:94 nickname:master no-dns:true no-masq-local:true port:6783]\n","stream":"stderr","time":"2020-09-08T07:46:28.659121082Z"}
{"log":"INFO: 2020/09/08 07:46:28.659062 weave  2.7.0\n","stream":"stderr","time":"2020-09-08T07:46:28.659187782Z"}
{"log":"INFO: 2020/09/08 07:46:29.029848 Bridge type is bridged_fastdp\n","stream":"stderr","time":"2020-09-08T07:46:29.06276161Z"}
{"log":"INFO: 2020/09/08 07:46:29.029915 Communication between peers is unencrypted.\n","stream":"stderr","time":"2020-09-08T07:46:29.06279591Z"}
{"log":"INFO: 2020/09/08 07:46:29.041053 Our name is de:60:e9:1e:f8:94(master)\n","stream":"stderr","time":"2020-09-08T07:46:29.062802611Z"}
{"log":"INFO: 2020/09/08 07:46:29.041123 Launch detected - using supplied peer list: [10.1.11.8 10.1.11.9]\n","stream":"stderr","time":"2020-09-08T07:46:29.062808911Z"}
{"log":"INFO: 2020/09/08 07:46:29.072615 Unable to fetch ConfigMap kube-system/weave-net to infer unique cluster ID\n","stream":"stderr","time":"2020-09-08T07:46:29.075112177Z"}
{"log":"INFO: 2020/09/08 07:46:29.072670 Using \"no-masq-local\" LocalRangeTracker\n","stream":"stderr","time":"2020-09-08T07:46:29.075145777Z"}
{"log":"INFO: 2020/09/08 07:46:29.072681 Checking for pre-existing addresses on weave bridge\n","stream":"stderr","time":"2020-09-08T07:46:29.075154377Z"}
{"log":"INFO: 2020/09/08 07:46:29.100509 [allocator de:60:e9:1e:f8:94] No valid persisted data\n","stream":"stderr","time":"2020-09-08T07:46:29.101404119Z"}
{"log":"INFO: 2020/09/08 07:46:29.145766 [allocator de:60:e9:1e:f8:94] Initialising via deferred consensus\n","stream":"stderr","time":"2020-09-08T07:46:29.14591046Z"}
{"log":"INFO: 2020/09/08 07:46:29.146053 Sniffing traffic on datapath (via ODP)\n","stream":"stderr","time":"2020-09-08T07:46:29.146146161Z"}
{"log":"INFO: 2020/09/08 07:46:29.156779 -\u003e[10.1.11.8:6783] attempting connection\n","stream":"stderr","time":"2020-09-08T07:46:29.156936419Z"}
{"log":"INFO: 2020/09/08 07:46:29.156950 -\u003e[10.1.11.9:6783] attempting connection\n","stream":"stderr","time":"2020-09-08T07:46:29.15712742Z"}
{"log":"INFO: 2020/09/08 07:46:29.161470 -\u003e[10.1.11.9:6783|1e:74:18:cc:f9:3b(worker2)]: connection ready; using protocol version 2\n","stream":"stderr","time":"2020-09-08T07:46:29.161617745Z"}
{"log":"INFO: 2020/09/08 07:46:29.161579 overlay_switch -\u003e[1e:74:18:cc:f9:3b(worker2)] using fastdp\n","stream":"stderr","time":"2020-09-08T07:46:29.161709045Z"}
{"log":"INFO: 2020/09/08 07:46:29.163643 -\u003e[10.1.11.9:6783|1e:74:18:cc:f9:3b(worker2)]: connection added (new peer)\n","stream":"stderr","time":"2020-09-08T07:46:29.163767956Z"}
{"log":"INFO: 2020/09/08 07:46:29.165242 Listening for HTTP control messages on 127.0.0.1:6784\n","stream":"stderr","time":"2020-09-08T07:46:29.165341365Z"}
{"log":"INFO: 2020/09/08 07:46:29.168886 Listening for metrics requests on 0.0.0.0:6782\n","stream":"stderr","time":"2020-09-08T07:46:29.169171585Z"}
{"log":"INFO: 2020/09/08 07:46:29.180259 -\u003e[10.1.11.8:6783|26:fa:6a:2a:3a:ca(worker1)]: connection ready; using protocol version 2\n","stream":"stderr","time":"2020-09-08T07:46:29.180420846Z"}
{"log":"INFO: 2020/09/08 07:46:29.181199 overlay_switch -\u003e[26:fa:6a:2a:3a:ca(worker1)] using fastdp\n","stream":"stderr","time":"2020-09-08T07:46:29.181282151Z"}
{"log":"INFO: 2020/09/08 07:46:29.181306 -\u003e[10.1.11.8:6783|26:fa:6a:2a:3a:ca(worker1)]: connection added (new peer)\n","stream":"stderr","time":"2020-09-08T07:46:29.181383751Z"}
(중략)
```

#### Log의 제목에 담긴 의미!

* 제목
```
weave-net-q6vj7_kube-system_weave-npc-07970c3cdb162d428ba13f7be372847284b8e567f16b3eaeb05e59a4fc62d0bc.log -> /var/log/pods/kube-system_weave-net-q6vj7_fddb8017-e841-46dc-9642-7b20a99f7078/weave-npc/0.log
```

* `weave-net-q6vj7`: Pod 명
* `kube-system`: Namespace 명
* `weave-npc-07970c3cdb162d428ba13f7be372847284b8e567f16b3eaeb05e59a4fc62d0bc`: Container 명