# Network

### 쿠버네티스 네트워크 모델

1. 한 Pod에 있는 다수의 Container간의 통신
2. Pod간의 통신
3. Pod와 Service 사이의 통신
4. 외부 클라이언트와 Service 사이의 통신

# 1. 한 Pod에 있는 다수의 Container간의 통신

### 특징

- 인터페이스를 공유
- `pause` 명령을 실행해 아무 동작을 하지 않는 빈 Container 생성 -> 주 Container에 문제가 생겨서 껐다 켜져도 공유 인터페이스가 유지되게 하기 위해서
- Port를 겹치게 구성하면 충돌 발생

![](/k8s-core-concepts/images/12-Network-1.png)
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

- eth0: 물리 인터페이스
- docker0: Docker를 설치하면 생기는 인터페이스
- veth: Container or Pod 만들면 생기는 인터페이스

### `pause` 관찰

Docker의 기능을 사용해서 Kubernetes Container를 관찰하면 다음과 같이 각 Pod마다 하나의 pause image를 실행시키고 있는 것을 볼 수 있다.

```
$ sudo docker ps | grep pause
5ecc86549782        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_weave-net-q6vj7_kube-system_fddb8017-e841-46dc-9642-7b20a99f7078_0
3306e3417288        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_kube-proxy-49zfz_kube-system_b31b76d8-db32-44f6-9d78-6b81c9ee5391_0
2f8c5dee3af0        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_kube-scheduler-master_kube-system_5146743ebb284c11f03dc85146799d8b_0
9da04bd3d3ad        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_kube-controller-manager-master_kube-system_6364e43ef3555c356a1732732a00b65f_0
0db2a0388bf3        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_kube-apiserver-master_kube-system_096e7adfe6a9e2c49849c30fd9cabd8a_0
1fe5fc9cac6c        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                               k8s_POD_etcd-master_kube-system_5f1924a86339fb52ac1fc0ed5c10b18e_0
```

주 Container들이 인터페이스를 사용하는 동안, 가만히 멈춰있는(?) 역할을 한다. 아무것도 하지 않지만 존재하므로 인터페이스가 유지되게 한다(= 인터페이스가 유지되려면 Pod가 하나 필요함).

* 예시: `api-server` 관찰

`master` 서버에서 동작 중인 apiserver Container를 관찰해보면 `kube-apiserver` Image와 `pause` Image가 동작하고 있다.

```
$ sudo docker ps -a | grep apiserver
e6bffcd1bbb9        1b74e93ece2f            "kube-apiserver --ad…"   22 hours ago        Up 22 hours                                     k8s_kube-apiserver_kube-apiserver-master_kube-system_096e7adfe6a9e2c49849c30fd9cabd8a_1
0db2a0388bf3        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                                       k8s_POD_kube-apiserver-master_kube-system_096e7adfe6a9e2c49849c30fd9cabd8a_0
```


* 예시: `kube-proxy` 관찰

`master` 서버에서 동작 중인 kubeproxy Container를 관찰해보면 `kube-proxy` Image와 `pause` Image가 동작하고 있다.

```
$ sudo docker ps -a | grep kube-proxy
dcaf6a1fb45e        bc9c328f379c            "/usr/local/bin/kube…"   3 days ago          Up 3 days                                       k8s_kube-proxy_kube-proxy-49zfz_kube-system_b31b76d8-db32-44f6-9d78-6b81c9ee5391_0
3306e3417288        k8s.gcr.io/pause:3.2    "/pause"                 3 days ago          Up 3 days                                       k8s_POD_kube-proxy-49zfz_kube-system_b31b76d8-db32-44f6-9d78-6b81c9ee5391_0
```

### `ifconfig`로 네트워크 관찰

- docker0
- eth0
- vethew-{random value}: 컨테이너들이 사용하고 있는 인터페이스!!

```
$ ifconfig
datapath: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1376
        inet6 fe80::3887:d6ff:fe62:7213  prefixlen 64  scopeid 0x20<link>
        ether 3a:87:d6:62:72:13  txqueuelen 1000  (Ethernet)
        RX packets 912  bytes 76843 (76.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 105  bytes 7446 (7.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:91:b7:be:2c  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.11.7  netmask 255.255.255.0  broadcast 10.1.11.255
        inet6 fe80::20d:3aff:fe88:12d5  prefixlen 64  scopeid 0x20<link>
        ether 00:0d:3a:88:12:d5  txqueuelen 1000  (Ethernet)
        RX packets 5316093  bytes 3483812801 (3.4 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3973638  bytes 1717414915 (1.7 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 69643110  bytes 16451651819 (16.4 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 69643110  bytes 16451651819 (16.4 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethwe-bridge: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1376
        inet6 fe80::9479:8dff:fef7:57f0  prefixlen 64  scopeid 0x20<link>
        ether 96:79:8d:f7:57:f0  txqueuelen 0  (Ethernet)
        RX packets 1168  bytes 125414 (125.4 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 209  bytes 18734 (18.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethwe-datapath: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1376
        inet6 fe80::4810:30ff:fe8a:b600  prefixlen 64  scopeid 0x20<link>
        ether 4a:10:30:8a:b6:00  txqueuelen 0  (Ethernet)
        RX packets 209  bytes 18734 (18.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1168  bytes 125414 (125.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vxlan-6784: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 65535
        inet6 fe80::c4cd:51ff:fe60:4e2a  prefixlen 64  scopeid 0x20<link>
        ether c6:cd:51:60:4e:2a  txqueuelen 1000  (Ethernet)
        RX packets 67631  bytes 91701996 (91.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 67024  bytes 91628913 (91.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

weave: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1376
        inet 10.38.0.0  netmask 255.240.0.0  broadcast 10.47.255.255
        inet6 fe80::dc60:e9ff:fe1e:f894  prefixlen 64  scopeid 0x20<link>
        ether de:60:e9:1e:f8:94  txqueuelen 1000  (Ethernet)
        RX packets 1126  bytes 104909 (104.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 204  bytes 18288 (18.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```


# 2. Pod간의 통신

### 특징

- Pod간 통신을 위해서는 CNI 플러그인이 필요
    - ACI, AOS, AWS VPN CNI, CNI-Genie, GCE, flannel, Weave Net, Calico등

- Weave Net
![](/k8s-core-concepts/images/12-Network-2.png)
이미지 출처: https://www.objectif-libre.com/en/blog/2018/07/05/k8s-network-solutions-comparison/

- CNI 비교

![](/k8s-core-concepts/images/12-Network-3.png)

![](/k8s-core-concepts/images/12-Network-4.png)

- Support Matrix

![](/k8s-core-concepts/images/12-Network-5.png)

![](/k8s-core-concepts/images/12-Network-6.png)

이미지 출처: https://chrislovecnm.com/kubernetes/cni/choosing-a-cni-provider/

- CNI 플러그인 지원 기능 설명

| 기능 | 설명 | 
| --- | --- |
| Network Model | * VXLAN: 캡슐화된 네트워킹으로 이론적으로 속도가 느림<br/>* Layer2: 캡슐화되지 않았기 때문에 Overhead의 영향이 없음|
| Route Distribution | * BGP 프로토콜 사용<br/>* 네트워크 세그먼트에 분할 된 클러스터를 구축하려는 경우 사용<br/>* 인터넷에서 라우팅 및 연결 가능성 정보를 교환하도록 설계된 외부 게이트웨이 프로토콜|
| Network Ploicies | 네트워크 정책을 사용하여 Pod가 서로 통신 할 수 있는 규칙을 적용 |
| Mesh Networking | Kubernetes 클러스터간 "Pod to Pod" 네트워킹이 가능  |
| Encyption | 네트워크 컨트롤 플레인을 암호화하여 모든 TCP 및 UDP 트래픽을 암호화 |
| Ingress / Egress Policies| 들어오가나 나가는 패킷에 대한 통신 제어 |

### Pod간 통신 확인

* 프로세스 및 Pod 확인
```
$ sudo netstat -antp | grep weave
tcp        0      0 127.0.0.1:6784          0.0.0.0:*               LISTEN      23755/weaver
tcp        0      0 10.1.11.7:54003         10.1.11.9:6783          ESTABLISHED 23755/weaver
tcp        0      0 10.1.11.7:35648         10.96.0.1:443           ESTABLISHED 24055/weave-npc
tcp        0      0 10.1.11.7:51811         10.1.11.8:6783          ESTABLISHED 23755/weaver
tcp6       0      0 :::6781                 :::*                    LISTEN      24055/weave-npc
tcp6       0      0 :::6782                 :::*                    LISTEN      23755/weaver
tcp6       0      0 :::6783                 :::*                    LISTEN      23755/weaver
```

위의 `10.1.11.7:54003`와 `10.1.11.7:51811`가 다른 Node의 Pod와도 통신하기 위해 연결된 DeamonSet으로 띄워진 인터페이스이다.
```
tcp        0      0 10.1.11.7:54003         10.1.11.9:6783          ESTABLISHED 23755/weaver
tcp        0      0 10.1.11.7:51811         10.1.11.8:6783          ESTABLISHED 23755/weaver
```

위의 PID로 프로세스를 조회하면 `--expect-npc --no-masq-local 10.1.11.8 10.1.11.9`: 어떤 Node와 연결되었는지 확인 가능하다.
```
$ ps -eaf | grep 23755
root      23755  23665  0 Sep08 ?        00:04:00 /home/weave/weaver --port=6783 --datapath=datapath --name=de:60:e9:1e:f8:94 --host-root=/host --http-addr=127.0.0.1:6784 --metrics-addr=0.0.0.0:6782 --docker-api= --no-dns --db-prefix=/weavedb/weave-net --ipalloc-range=10.32.0.0/12 --nickname=master --ipalloc-init consensus=2 --conn-limit=200 --expect-npc --no-masq-local 10.1.11.8 10.1.11.9
ldccai   106365  37265  0 06:22 pts/1    00:00:00 grep --color=auto 23755
```

* 참고: DeamonSet

Node마다 하나씩 Pod를 배치하는 방식으로 weave Net에 대한 설정 파일 조회하면 다음과 같다.
```
$ kubectl get ds weave-net -n kube-system -o yaml
apiVersion: apps/v1
kind: DaemonSet

(중략)
      volumes:
      - hostPath:
          path: /var/lib/weave      <-- Node의 경로와 volume 마운트 볼 수 있음
          type: ""
        name: weavedb
      - hostPath:
          path: /opt
          type: ""
        name: cni-bin
      - hostPath:
          path: /home
          type: ""
        name: cni-bin2
      - hostPath:
          path: /etc
          type: ""
        name: cni-conf
      - hostPath:
          path: /var/lib/dbus
          type: ""
        name: dbus
      - hostPath:
          path: /lib/modules
          type: ""
        name: lib-modules
      - hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
        name: xtables-lock
(중략)
```

# 3. Pod와 Service 사이의 통신

### 특징

- iptables는 리눅스 커널 기능인 netfilter를 사용하여 network traffic 제어

![](/k8s-core-concepts/images/12-Network-7.png)
이미지 출처: https://kubernetes.io/docs/concepts/services-networking/service/

- kune-proxy라는 컴포넌트로 서비스 트래픽을 제어

![](/k8s-core-concepts/images/12-Network-8.png)
이미지 출처: hhttps://ko.wikipedia.org/wiki/Iptables/

### 서비스 IP를 통해 통신하는 흐름

다음 그림은 서비스 IP를 통해 10.3.241.152에 요청하는 흐름을 나타내고 있다.

![](/k8s-core-concepts/images/12-Network-9.png)
![](/k8s-core-concepts/images/12-Network-10.png)

이미지 출처: https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82

### iptable에서 목록 확인

Cluster를 생성하면 iptables의 설정이 적용된다.

* ClusterIP 확인(Kubernetes cluster를 구성하면 자동으로 생성)

```
$ kubectl get svc --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  4d
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4d
```

* ClusterIP에 등록된 iptables chain 정책 확인

```
ldccai@master:~$ sudo iptables -S -t nat | grep 10.96
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:metrics cluster IP" -m tcp --dport 9153 -j KUBE-SVC-JD5MR3NA4I4DYORP
-A KUBE-SERVICES -d 10.96.0.1/32 -p tcp -m comment --comment "default/kubernetes:https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y
-A KUBE-SERVICES -d 10.96.0.10/32 -p udp -m comment --comment "kube-system/kube-dns:dns cluster IP" -m udp --dport 53 -j KUBE-SVC-TCOU7JCQXEZGVUNU
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns:dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
```

# 4. 외부 클라이언트와 Service 사이의 통신

### 특징

- netfilter와 kube-proxy 기능을 사용해 원하는 Service 및 Pod로 연결

![](/k8s-core-concepts/images/12-Network-11.png)

이미지 출처: https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82