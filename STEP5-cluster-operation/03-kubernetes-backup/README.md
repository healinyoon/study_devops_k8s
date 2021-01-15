Kubernetes 백업 및 복원 방법에 대해 알아본다.

# 백업 목록

* Pod 정보 YAML 파일
* Etcd 데이터베이스
* PV

각각의 백업 방법을 하나씩 살펴보자.

# Pod 정보 YAML 파일 백업 / 복원

Deployment, ReplicaSet, Service, Pod 등에 해당된다. 

### 백업

Kubernetes에서 동작하는 모든 resource를 `-o yaml` 옵션을 사용하여 출력하여 백업한다.

```
$ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

### 복원

복원시에는 일반적인 YAML 파일로 resource를 실행하는 방법을 적용하면 된다.

```
$ kubectl create -f all-deploy-services.yaml
```

# Etcd 데이터베이스 백업 / 복원

ConfigMap, Secret, PVC 등에 해당된다. Etcd를 백업한다([ETCD 백업 공식 문서](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)). 

### 백업

> (참고) Etcd를 백업하려면 ETCD가 당연히 먼저 설치되어 있었어야 한다.

```
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save {snapshot name}
# exit 0

# verify the snapshot
ETCDCTL_API=3 etcdctl --write-out=table snapshot status {snapshot name}
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |       10 |          7 | 2.1 MB     |
+----------+----------+------------+------------+
```

### 복원

복원시 사용되는 옵션에 대한 상세 경로는 [옵션에 대한 상세 정보](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/configuration.md)를 참고한다. 복원시 아래 절차를 순서대로 수행한다.

#### 1. 복원 명령어 실행

```
sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd-restore \
--initial-cluster='master=https://127.0.0.1:2380' \
--name=master \
--initial-cluster-token {token} \
--initial-advertise-peer-urls https://127.0.0.1:2380 \
snapahot restore {snapshot 경로}
```

#### 2. etcd.yaml StaticPod 수정

위의 복원 명령어에서 변경된 부분이 있기 때문에 StaticPod도 수정해준다.

```
# sudo vi /etc/kubernetes/manifests/etcd.yaml

① 아래의 설정 값을 모두 찾아서 수정

수정 전)
/var/lib/etcd

수정 후)
/var/lib/etcd-restore

② 옵션 추가
--initial-cluster-token={token}
```

#### 3. Etcd가 부팅되고 1분정도 기다린 후 kubectl 동작을 확인한다.

그래도 정상 동작하지 않을 경우 docker log를 조회해서 문제의 원인을 파악하고 해결해야 한다.

# Persistent Volume 백업 / 복원

일반적인 방법으로 백업한다.
