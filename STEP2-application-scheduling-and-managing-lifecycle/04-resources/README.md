cpu m 단위, mem mi 단위로 많이 사용 => 작은 단위의 컨테이너를 여러개 사용하는 것을 지향

$ kubectl create -f nginx-deploy.yaml
deployment.apps/nginx created

$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
nginx-7b47bbb85b-4vdrm     1/1     Running   0          19s
nginx-7b47bbb85b-ldtdk     1/1     Running   0          19s
nginx-7b47bbb85b-q8vw2     1/1     Running   0          19s

$ kubectl get pod nginx-7b47bbb85b-4vdrm -o yaml

# limitRanges

[※ 쿠버네티스 LimitRange 공식 문서](https://kubernetes.io/docs/concepts/policy/limit-range/)

Namespace에서 Pod 또는 Container 별로 리소스를 제한하는 정책이다.  
Apiserver 옵션에 `--enable-admission-plugins=LimitRange`를 설정하여 사용한다.

### limitRange의 기능
* Namespace에서 Pod나 Container 당 최소 및 최대 컴퓨팅 리소스 사용량 제한
* Namespace에서 PersistentVolumeClaim 당 최소 및 최대 스토리지 사용량 제한
* Namespace에서 리소스에 대한 요청(request)과 제한(limit) 사이의 비율 적용
* Namespace에서 컴퓨팅 리소스에 대한 default requests/limit을 설정하고, 런타임 중인 컨테이너에 자동으로 입력


### 3가지 방법으로 가능

* Container 수준의 리소스 제한
* Pod 수준의 리소스 제한
* 스토리지 리소스 제한

# 실습

### api-server에 설정 추가

`--enable-admission-plugins=LimitRange` 설정

```
$ sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

아래 내용을
- --enable-admission-plugins=NodeRestriction

다음과 같이 수정
- --enable-admission-plugins=NodeRestriction,LimitRanger
```

### (실습 환경) Namespace 생성
```
$ kubectl create ns limitrage
namespace/limitrage created
```

### 1. Container 수준의 리소스 제한

제한하기 원하는 Namespace에 LimitRange 리소스를 생성한다. 

* LimitRange YAML 실행
> limit-mem-cpu-per-container.yaml

```
$ kubectl create -f https://k8s.io/examples/admin/resource/limit-mem-cpu-container.yaml -n ns-limitrange
limitrange/limit-mem-cpu-per-container created
```

* 제한 설명

| 기준 | CPU | MEM |
| --- | --- | --- |
| 최대 | 800m | 1Gi |
| 최소 | 100m | 99Mi |
| 기본 제한 | 700m | 900Mi |
| 기본 요구 사항 | 110m | 111Mi |

* LimitRange 조회
```
$ kubectl describe limitrange -n ns-limitrange
Name:       limit-mem-cpu-per-container
Namespace:  ns-limitrange
Type        Resource  Min   Max   Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---   ---------------  -------------  -----------------------
Container   cpu       100m  800m  110m             700m           -
Container   memory    99Mi  1Gi   111Mi            900Mi          -
```

* 리소스를 사용하는 Pod YAML 실행
```
$ kubectl apply -f https://k8s.io/examples/admin/resource/limit-range-pod-1.yaml -n ns-limitrange
pod/busybox1 created
```

* Pod 조회
```
$ kubectl get pod -n ns-limitrange busybox1
NAME       READY   STATUS    RESTARTS   AGE
busybox1   4/4     Running   0          6m27s
```

* Pod 상세 조회로 리소스 설정 확인
```
$ kubectl get pod -n ns-limitrange busybox1 -o yaml

(중략)
spec:
  containers:
  - args:
    - -c
    - while true; do echo hello from cnt01; sleep 10;done
    command:
    - /bin/sh
    image: busybox
    imagePullPolicy: Always
    name: busybox-cnt01
    resources:								<-- limit과 reqeust를 모두 설정해준 경우 설정한대로 생성된다.
      limits:
        cpu: 500m
        memory: 200Mi
      requests:
        cpu: 100m
        memory: 100Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-rp4wv
      readOnly: true
  - args:
    - -c
    - while true; do echo hello from cnt02; sleep 10;done
    command:
    - /bin/sh
    image: busybox
    imagePullPolicy: Always
    name: busybox-cnt02
    resources:								<-- request만 주고, limit을 주지 않는 경우 기본 limit을 사용
      limits:
        cpu: 700m
        memory: 900Mi
      requests:
        cpu: 100m
        memory: 100Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-rp4wv
      readOnly: true
  - args:
    - -c
    - while true; do echo hello from cnt03; sleep 10;done
    command:
    - /bin/sh
    image: busybox
    imagePullPolicy: Always
    name: busybox-cnt03
    resources:                                                          <-- limit은 주고 reqeust를 주지 않는 경우, 기본 reqeust를 사용
      limits:
        cpu: 500m
        memory: 200Mi
      requests:
        cpu: 500m
        memory: 200Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-rp4wv
      readOnly: true
  - args:
    - -c
    - while true; do echo hello from cnt04; sleep 10;done
    command:
    - /bin/sh
    image: busybox
    imagePullPolicy: Always
    name: busybox-cnt04
    resources:                                                          <-- request와 limit 모두 안주는 경우, 각각 기본 요청과 제한을 따른다.
      limits:
        cpu: 700m
        memory: 900Mi
      requests:
        cpu: 110m
        memory: 111Mi
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-rp4wv
      readOnly: true
(중략)
```

### 리소스 제한을 초과하는 Container를 띄우는 경우

* 리소스 제한을 초과하는 Container를 띄우는 Deployment YAML 작성

> nginx-deploy-over-resource.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    run: nginx
  namespace: ns-limitrange
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
       run: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "200Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
status: {}
```

* YAML 실행
```
$ kubectl create -f nginx-deploy-over-resource.yaml
deployment.apps/nginx created
```

* Deploy 조회

다음과 같이 Deploy가 아예 할당받지 못함
```
$ kubectl get deploy -n ns-limitrange
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/3     0            0           85s
```

Deployment 상세 조회로 `Events`를 확인해보면 다음과 같음
```
$ kubectl describe deploy -n ns-limitrange

(중략)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  101s  deployment-controller  Scaled up replica set nginx-7d67999896 to 3
```

위의 로그에서 replica 중에 중단된 것을 볼 수 있다 => 이번에는 replicaSet을 조회해서 살펴보자

* Replica 상세 조회
```
$ kubectl describe rs -n ns-limitrange

(중략)
Events:
  Type     Reason        Age                   From                   Message
  ----     ------        ----                  ----                   -------
  Warning  FailedCreate  4m33s                 replicaset-controller  Error creating: pods "nginx-7d67999896-jgck7" is forbidden: maximum cpu usage per Container is 800m, but limit is 1
```

위와 같이 maximum cpu를 초과해서 replicaSet 생성이 불가능하다는 로그를 볼 수 있다!


### 2. Pod 수준의 리소스 제한

제한하기 원하는 Namespace에 limitRange 리소스를 생성한다.

* LimitRange YAML 실행
> limit-mem-cpu-per-pod.yaml

* 제한 설명

| 기준 | CPU | MEM |
| --- | --- | --- |
| 최대 | 2 | 2Gi |
| 최소 | - | - |
| 기본 | - | - |
| 최소 요구 사항 | - | - |

* 리소스 조회
```
```

### 3. 스토리스 리소스 제한

제한하기 원하는 Namespace에 LimitRange 리소스를 생성한다.

* LimitRange YAML 작성
> limit-storages.yaml


* 제한 설명

| 기준 | 용량 |
| --- | --- | 
| 최대 | 2Gi |
| 최소 | 1Gi |

* 리소스 조회
```
```

# ResourceQuata

[※ 쿠버네티스 ResourceQuata 공식 문서](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)

### 개요

Namespace별 리소스의 "총량"을 제한한다.

* 예시

> mem-cpu-resource-quota.yaml
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-resource-quota
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: "2Gi"
```

* 예시 해석

Namespace 내의 모든 Container의 리소스 합이 다음을 넘어서는 안된다.

| 기준 | CPU | MEM |
| --- | --- | --- |
| 최대 | 2 | 2Gi |
| 최소 요구사항 | 1 | 1Gi |

### 실습

* namespace 생성
```
$ kubectl create namespace quota-mem-cpu-example
namespace/quota-mem-cpu-example created
```

* ResourceQuota YAML 실행
```
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu.yaml --namespace=quota-mem-cpu-example
resourcequota/mem-cpu-demo created
```

* ResourceQuota 조회
```
$ kubectl describe resourcequotas -n quota-mem-cpu-example
Name:            mem-cpu-demo
Namespace:       quota-mem-cpu-example
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     2
limits.memory    0     2Gi
requests.cpu     0     1
requests.memory  0     1Gi
```

* `quota-mem-cpu-example` Namespace의 resource를 사용하는 첫번째 Pod 생성
```
$ kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod.yaml --namespace=quota-mem-cpu-example
pod/quota-mem-cpu-demo created
```

* `quota-mem-cpu-example` Namespace의 resource 사용량 확인
```
$ kubectl describe resourcequotas -n quota-mem-cpu-example
Name:            mem-cpu-demo
Namespace:       quota-mem-cpu-example
Resource         Used   Hard
--------         ----   ----
limits.cpu       800m   2
limits.memory    800Mi  2Gi
requests.cpu     400m   1
```

* `quota-mem-cpu-example` Namespace의 resource를 사용하는 두번째 Pod 생성
```
$ kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod-2.yaml --namespace=quota-mem-cpu-example
Error from server (Forbidden): error when creating "https://k8s.io/examples/admin/resource/quota-mem-cpu-pod-2.yaml": pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo, requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi
```

위와 같은 에러 발생을 확인할 수 있다.

에러 내용을 살펴보면,  
`quests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi`  
Resource Quota의 설정을 초과하여 Pod 생성이 실패했음을 알 수 있다.


