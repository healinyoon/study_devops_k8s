Kubernetes 백업 및 복원을 실습한다.

# 1. 백업

YAML 백업 → Etcd 백업 → PV 백업을 순서대로 진행한다.

### 1.1. Pod 정보 YAML 파일 백업

모든 resource 정보를 YAML 파일로 출력한다.

```
$ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

모든 resource 정보를 포함하고 있는 File 라인 수는 무려 31310 이다.

```
$ cat all-deploy-services.yaml | wc -l
31310
```

### 1.2. Etcd 백업

#### 1.2.1 Etcd 백업 명령어 실행

Etcd 백업 명령어를 수행한다. 

* 형식: 

```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save {snapshot name}
```

`--cacert`, `--cert`, `--key` 옵션의 값은 `/etc/kubernetes/pki/etcd` 경로에 저장되어 있다.

* 예시: 

```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save snapshotdb
{"level":"info","ts":1610677780.6238728,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"snapshotdb.part"}
{"level":"info","ts":"2021-01-15T02:29:40.637Z","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1610677780.6374571,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"127.0.0.1:2379"}
{"level":"info","ts":"2021-01-15T02:29:40.779Z","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}
{"level":"info","ts":1610677781.0398917,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"127.0.0.1:2379","size":"7.4 MB","took":0.415933431}
{"level":"info","ts":1610677781.0400813,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"snapshotdb"}
Snapshot saved at snapshotdb
```

#### 1.2.2 Etcd 백업 결과 조회

위의 명령어를 통해 백업이 완료된 후에, 백업된 상태 정보를 조회하고 싶다면 다음 명령어를 사용하자.

* 형식:

```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status {snapshot name}
```

* 예시: 

```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status snapshotdb
2e633e9e, 27325346, 1340, 7.4 MB
```

옵션을 추가하면 table 형식으로 출력할 수도 있다.
```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status snapshotdb --write-out=table
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 2e633e9e | 27325346 |       1340 |     7.4 MB |
+----------+----------+------------+------------+
```

ETCD snapshot을 다른 서버에 백업하고 싶으면 `snapshotdb`을 복제해서 옮기면 된다.

### 1.3. PV 백업

PV는 각 PV의 유형별로 적절하게 백업해주면 된다.

# 2. 복원

Etcd 백업 → PV 백업 → YAML 백업을 순서대로 진행한다.

### 2.1. Etcd 복원

#### 2.1.1. snapahot 가져오기

미리 백업해둔 Etcd snapshot를 복사해온다.

#### 2.1.2. Etcd 복원
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

추가된 옵션을 살펴보면 다음과 같다.

* 디렉토리 변경(etcd → etcd-restore)
```
--data-dir /var/lib/etcd-restore \
```

* master server 정보 전달
```
--initial-cluster='master=https://127.0.0.1:2380' \
```

* master server 이름 전달
```
--name=master \
```

* token 정보
```
--initial-cluster-token {token} \
```

*  cluster가 여러개 Node로 구성된 경우, 다수의 서버를 입력한다.
```
--initial-advertise-peer-urls https://127.0.0.1:2380 \
```

#### 2.1.3. 변경된 restore 확인

위의 명령어를 통해 새롭게 생성된 `etcd-restore` 디렉토리를 확인한다.

```
$ sudo ls /var/lib/etcd-restre/
member
```

### 2.2. Pod 정보 YAML 파일 복원

#### 1. all-deploy-services.yaml 가져오기

#### 2. Resource 생성




