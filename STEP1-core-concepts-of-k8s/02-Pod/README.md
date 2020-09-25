# Pod

### 개요

* Conatiner의 그룹
* Kubernetes는 Container를 개별적으로 배포하지 않고, Container의 Pod를 항상 배포하고 운영한다.
  * 일반적으로 Pod는 단일 Container만 포함하지만 결합성이 강한 다수의 Container를 묶어서 구성할 수 있다.
* Pod는 다수의 Node에 걸쳐 배치될 수 없으므로 단일 Node에서만 실행된다.

### Pod 관리

#### Pod의 장점

* pod는 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보인다.
* 그러나 동일한 환경을 제공하면서 다소 격리된 상태로 유지된다.

#### 동일한 Pod 내에서 Container 사이의 부분 격리

* Pod의 모든 Container는 동일한 네트워크 및 UTS Namespace에서 실행된다.
    * UTS Namespace: namespace 별로 호스트명이나 도메인 명을 독자적으로 가질 수 있다. 즉 namespace 별로 hostname을 변경하고 분할 가능하다.
    * 따라서 같은 호스트 이름 및 네트워크 인터페이스를 공유 => 내부에는 port 충돌 가능성이 있다.
* Pod의 모든 Container는 동일한 IPC Namespace 아래에서 실행되며, IPC를 통해 통신 가능하다.
    * IPC Namespace: 프로세스간 통신(IPC = Inter-Process Communication = 프로세스간 서로 데이터를 주고 받는 경로, 대표적으로 signal, socket, pipe 등) 오브젝트를 Namespace 별로 독립적으로 가질 수 있다. 즉 namespace 별로 프로세스간 통신을 격리할 수 있다.


### Pod 구조

* Kubernetes cluster의 모든 Pod는 공유된 단일 플랫, 네트워크 주소 공간에 위치한다.
* Pod 사이에는 NAT 게이트웨이가 존재하지 않는다.

![](/STEP1-core-concepts-of-k8s/images/c)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

### 1 Container : 1 Pod ?

아래의 사항을 기준으로 적절한 구성이 필요하다.
* 다수의 Pod로 멀티 티어 애플리케이션 분할하기
* 각각 스케일링이 가능한 Pod로 분할하기

![](/STEP1-core-concepts-of-k8s/images/02-Pod-2.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

# Pod YAML 작성하기
* 모든 API에 대한 내용은 http://kubernetes.io/docs/reference/ 를 참고한다.

![](/STEP1-core-concepts-of-k8s/images/02-Pod-3.png)  

### Pod YAML 구성 요소

* Pod YAML은 apiVersion, kind, metadata, spec, status 등의 오브젝트로 구성된다.
  * apiVersion: k8s API의 버전을 가리킨다.
  * kind: 어떤 리소스 유형(Pod, ReplicaSet, Service, Deployment 등)인지 결정한다.
  * metadata: Pod와 관련된 정보(이름, Namespace, Label 등)가 존재한다.
  * spec: Pod내의 Container와 Container의 Volume 등의 정보가 존재한다.
  * status: Pod의 상태, 각 Container의 설명 및 상태, Pod 내부의 IP 및 그 밖의 기본 정보 등이 존재한다.
    * status는 k8s가 알아서 해준다.

### Pod YAML 작성 방법

* pod 디스크립터는 https://kubernetes.io/docs/concepts/workloads/pods/ 도큐먼트 참조  
(쿠버네티스 API가 버전업 되기 때문에 여기서 가져오는게 제일 정확함)

* 다음과 같이 간단히 작성

> http-go-pod.yaml
```
# 이 디스크립터는 k8s API v1을 사용
apiVersion: v1
# 리소스 종류: Pod
kind: Pod
# Pod의 정보
metadata:
  # Pod 명
  name: http-go
spec:
  containers:
  # Container 정보
  - name: http-go
    image: healinyoon/http-go
    ports:
    # 응답 대기할 애플리케이션 Port
    - containerPort: 8080
      protocol: TCP
```

### Pod YAML 작성 요령 확인 명령어

```
$ kubectl explain pods
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

### Pod 명령어

* 작성한 Pod YAML 실행
```
$ kubectl create -f {YAML 파일명}
```

* 실행한 Pod 조회
```
$ kubectl get pod
```

* 실행 중인 Pod 로그 읽기
```
$ kubectl logs {Pod 명}
```

* 실행 중인 Pod 삭제
```
$ kubectl delete pod {Pod 명}
```

* 실행 중인 모든 Pod 삭제
```
$ kubectl delete pod --all
```

### Pod의 정의된 YAML 읽어오는 방법

* 형식
```
$ kubectl get pod {Pod 명} -o yaml
```

* 예시
```
$ kubectl get pod http-go-5c6f458dc9-m97w8 -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-14T04:48:10Z"
  generateName: http-go-5c6f458dc9-
  labels:
    app: http-go
    pod-template-hash: 5c6f458dc9
(중략)
spec:
  containers:
  - image: healinyoon/http-go
    imagePullPolicy: Always
    name: http-go
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-fvtzt
      readOnly: true
(중략)
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-09-14T04:48:10Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-09-14T04:48:14Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-09-14T04:48:14Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-09-14T04:48:10Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://3d6f295961dc40e90bb4c5dd7258c0c24b7cb723b758216107183f337d1a171e
    image: healinyoon/http-go:latest
    imageID: docker-pullable://healinyoon/http-go@sha256:8117434d391b65a517b690295b9f0780ba3ffaf2625048c45327fc4f3b739243
    lastState: {}
    name: http-go
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-09-14T04:48:14Z"
```

# Pod YAML 더 알아보기

### Container에서 Host로 port-forwarding

* 디버깅 혹은 다른 이유로 Service 리소스를 거치지 않고 특정 Pod와 통신하고 싶을 때 사용한다.
* `kubectl port-forward` 명령으로 수행

> 예시: Container 8888 port -> Pod 8080 port로 전달
```
$ kubectl port-forward {Pod 명} {Container Port}:{Pod port}

$ kubectl port-forward http-go 8888:8080

-> curl로 확인
$ curl 127.0.0.1:8888
``` 

### Pod에 주석 추가하기

* 각 Pod나 API 객체에 주석을 추가할 수 있다.
* Cluster를 사용하는 모든 사람이 각 오브젝트의 정보를 빠르게 파악 가능하다.
  * 예를 들면 오브젝트를 생성한 사람의 이름을 지정
* 공동 작업이 가능하게 한다.
* 총 256KB까지 포함 가능하다.

#### 주석 추가 방법

* 주석 추가
```
$ kubectl annotate pod {Pod 명} key="value"
```

* 주석 확인
```
$ kubectl get pod {Pod 명} -o yaml
```

* 예시
```
$ kubectl annotate pod http-go-5c6f458dc9-m97w8 test="test1234"
pod/http-go-5c6f458dc9-m97w8 annotated

$ kubectl get pod http-go-5c6f458dc9-m97w8 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    test: test1234
  creationTimestamp: "2020-09-14T04:48:10Z"
  generateName: http-go-5c6f458dc9-
```