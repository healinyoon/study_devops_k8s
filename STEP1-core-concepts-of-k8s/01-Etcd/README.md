# ETCD

[※ ETCD Git 리포지토리](https://github.com/etcd-io/etcd)

"A distributed, reliable key-value store for the most critical data of a distributed system."

### 개요

ETCD는 고가용성을 제공하는 분산형 key-value 저장소 오픈소스이며, Kubernetes에서 필요한 모든 데이터를 저장하는 실질적인 데이터베이스이다.   

* Key-Value Data Set
![](/STEP1-core-concepts-of-k8s/images/01-Etcd-1.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

* Multi Key-Value Data Set

value 접근 방법 예시: User[1]['Name']

![](/STEP1-core-concepts-of-k8s/images/01-Etcd-2.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

# Etcd 설치 및 사용법

### 설치(Master Node)

* 1. Google에 'Etcd github' 검색
![](/STEP1-core-concepts-of-k8s/images/01-Etcd-3.png)  

* 2. tar 파일 다운로드
![](/STEP1-core-concepts-of-k8s/images/01-Etcd-4.png)  

```
$ wget https://github.com/etcd-io/etcd/releases/download/v3.4.10/etcd-v3.4.10-linux-amd64.tar.gz
```

* 3. 압축 해제
```
$ tar -xf etcd-v3.4.10-linux-amd64.tar.gz
$ ls
etcd-v3.4.10-linux-amd64
```

#### Etcd 조회
```
$ ./etcd-v3.4.10-linux-amd64
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints  127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.admissionregistration.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.apiextensions.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.coordination.k8s.io
(중략)
```

위의 내용을 살펴보면 다음과 같다.

* `ETCDCTL_API=3` API 버전을 환경변수로 설정해두고 사용
* 출력 값: Etcd에 저장되는 정보들
```
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.admissionregistration.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.apiextensions.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.coordination.k8s.io
```

### Etcd 입력 및 출력

* 입력
```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints  127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key put key1 value1
OK
```

* 출력
```
$ sudo ETCDCTL_API=3 ./etcdctl --endpoints  127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get key1
key1
value1
```

# Kuberentes Etcd 데이터베이스 Key 구조(Multi Key-Value)

Etcd 내부에 Kuberentes의 전체 설정 정보가 저장된다.
![](/STEP1-core-concepts-of-k8s/images/01-Etcd-5.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터
