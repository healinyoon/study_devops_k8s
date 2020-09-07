# 롤링 업데이트와 롤백 실습 - 1

### Deployment 실행

```
$ kubectl create -f http-go-deploy-v1.yaml
deployment.apps/http-go created
```

### Deployment 확인

```
$ kubectl get all
NAME                          READY   STATUS    RESTARTS   AGE
pod/http-go-ccb794f48-ff26p   1/1     Running   0          19s
pod/http-go-ccb794f48-lvqjp   1/1     Running   0          19s
pod/http-go-ccb794f48-nlm8s   1/1     Running   0          19s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4m11s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/http-go   3/3     3            3           19s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-ccb794f48   3         3         3       19s
```

### Deployment의 상세 내용 확인

Replicas, StrategyType, Events 등을 확인 가능
```
$ kubectl describe deploy http-go
```

아래 내용이 출력됨
```
Name:                   http-go
Namespace:              default
CreationTimestamp:      Fri, 04 Sep 2020 05:43:12 +0000
Labels:                 app=http-go
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=http-go
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=http-go
  Containers:
   http-go:
    Image:        gasbugs/http-go:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   http-go-ccb794f48 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  112s  deployment-controller  Scaled up replica set http-go-ccb794f48 to 3
```

### Deployment yaml 파일을 확인하고 싶을 때

```
$ kubectl get deploy http-go -o yaml
```

아래 내용이 출력됨
```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2020-09-04T05:43:12Z"
  generation: 1
  labels:
    app: http-go
  
(중략)
  
  name: http-go
  namespace: default
  resourceVersion: "342721"
  selfLink: /apis/apps/v1/namespaces/default/deployments/http-go
  uid: 9311de35-631a-4acf-a5f2-6a314042ba8d
spec:
  progressDeadlineSeconds: 600
  replicas: 3                   # 개수
  revisionHistoryLimit: 10      # 업데이트 히스토리 기록 개수 => 추후 원하는 시점으로 백업 가능
  selector:
    matchLabels:
      app: http-go
  strategy:
    rollingUpdate:
      maxSurge: 25%             # 개수 or percentage로 입력 가능
      maxUnavailable: 25%
    type: RollingUpdate

(중략)
```

# 롤링 업데이트와 롤백 실습 - 2

애플리케이션 모니터링 시스템을 만들고, 실제로 업데이트시 애플리케이션이 무중단 되는지 관찰

### (실습을 위해) 기존의 모든 애플리케이션 제거
```
$ kubectl delete all --all
```

### Deployment 실행 

`--record=true` 옵션 사용 => 업데이트 히스토리 기록 가능 => 백업 가능
```
$ kubectl create -f http-go-deploy-v1.yaml --record=true
```

### Deploy status 확인
```
$ kubectl rollout status deploy http-go
deployment "http-go" successfully rolled out    # 업데이트가 끝나서 배포가 잘 되었다는 의미
```

### 히스토리 확인

`--recorde=true`를 주었기 때문에 가능, 그렇지 않을 경우 아래 명령어는 공백이 출력됨
```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deploy-v1.yaml --record=true
```

### patch 명령어로 minReadySeconds 입력

`patch` 명령어로 내부 설정(=yaml 파일)을 수정할 수 있음
```
$ kubectl patch deploy http-go -p '{"spec": {"minReadySeconds": 10}}'   # ready를 위한 시간으로 10초를 입력(안주면 굉장히 빨리 업데이트가 돼서 관찰용으로 입력)
deployment.apps/http-go patched
```

### 로드밸런서 생성 및 확인

서비스(로드밸런서) 생성
```
$ kubectl expose deploy http-go
service/http-go exposed
```

서비스 확인
```
$  kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
http-go      ClusterIP   10.107.154.244   <none>        8080/TCP   38s
```
http-go가 10.107.154.244에 오픈되어 있는 것 확인

### 업데이트할 때도 애플리케이션이 무중단인지 확인하기 위한 모니터링 프로그램 생성
```
$ kubectl run -it --rm --image busybox -- bash
If you don't see a command prompt, try pressing enter.
/ # wget -O- -q 10.107.154.244:8080
Welcome! v1
/ # while true; do wget -O- -q 10.107.154.244:8080; sleep 1; done    <-- 10초에 한번씩 출력
Welcome! v1     
Welcome! v1
Welcome! v1
```

### 업데이트 수행

`set images` 명령어를 이용한 업데이트 수행
```
$ kubectl set image deploy {deployment 명} {deployment 내의 container 명(container가 2개 이상일 수 있기 때문)}
$ kubectl set image deploy http-go http-go=gasbugs/http-go:v2
deployment.apps/http-go image updated
```

Pod를 확인해보면 Pod가 하나씩 변경된는 것을 확인할 수 있음
```
$ kubectl get pod -w
NAME                       READY   STATUS    RESTARTS   AGE
bash                       1/1     Running   0          9m18s
http-go-6c4d9d5989-lv6mt   1/1     Running   0          9s
http-go-6c4d9d5989-m9wqf   1/1     Running   0          27s
http-go-ccb794f48-bwb5v    1/1     Running   0          53m
http-go-ccb794f48-spxbj    1/1     Running   0          53m
http-go-ccb794f48-bwb5v    1/1     Terminating   0          53m
http-go-6c4d9d5989-zr58b   0/1     Pending       0          0s
http-go-6c4d9d5989-zr58b   0/1     Pending       0          0s
http-go-6c4d9d5989-zr58b   0/1     ContainerCreating   0          0s
http-go-ccb794f48-bwb5v    0/1     Terminating         0          53m
http-go-6c4d9d5989-zr58b   1/1     Running             0          2s
http-go-ccb794f48-bwb5v    0/1     Terminating         0          53m
http-go-ccb794f48-bwb5v    0/1     Terminating         0          53m
http-go-ccb794f48-spxbj    1/1     Terminating         0          53m
http-go-ccb794f48-spxbj    0/1     Terminating         0          53m
```

마찬가지로 모니터링 출력을 확인하면  

`Welcome! v1`을 출력하다가
```
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
```

`Welcome! v1`과 `Welcome! v2`가 함께 출력되가가
```
Welcome! v2
Welcome! v1
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
```

모든 업데이트가 완료되면 `Welcome! v2`만 출력되는 것을 볼 수 있음 
```
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
```

### !!! 주의: --record=true

위의 `$ kubectl set image deploy http-go http-go=gasbugs/http-go:v2` 명령에서 `--record=true`을 옵션으로 주지 않았기 때문에  
히스토리를 출력해보면 다음과 같이 방금 전의 업데이트가 기록되지 않음 => 백업 불가
```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deploy-v1.yaml --record=true
2         kubectl create --filename=http-go-deploy-v1.yaml --record=true

~~~ 방금 전 업데이트의 기록이 남지 않아있음 => 여전히 http-go의 버전이 v1인것 처럼 보임 ~~~
```

따라서 다시 `--record=true` 옵션과 함께 업데이트를 수행하면
```
$ kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
deployment.apps/http-go image updated
```

아래와 같이 업데이트 기록을 확인할 수 있음
```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deploy-v1.yaml --record=true
2         kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
```

그런데 왜 REVISION 번호가 바뀌지 않고 2번에 덮어쓰여졌는가? => replicas가 변경된 내용이 없었기 때문  
이게 무슨 소리냐면, 현재 relicaset을 출력해보면 다음과 같음
```
$ kubectl get rs
NAME                 DESIRED   CURRENT   READY   AGE
http-go-6c4d9d5989   3         3         3       11m
http-go-ccb794f48    0         0         0       64m
```

`http-go-ccb794f48`는 업데이트 전의 replicaset이고, `http-go-6c4d9d5989`는 업데이트한 replicaset임.  
즉, 업데이트시 기존의 replicaset을 제거하는 것이 아니라, **롤백을 위해** 기존 replicaset의 pod 개수를 0개로 줄여버리고 새로운 replicaset을 배포하는 것임


# 롤링 업데이트와 롤백 실습 - 3

이번에는 다른 방법으로 업데이트 진행

### 업데이트

`edit` 명령어를 사용하여 Deployment yaml 파일 수정
```
$ kubectl edit deploy http-go --record=true
deployment.apps/http-go edited
```

아래 image의 버전 수정(v2 -> v3)
```
    spec:
      containers:
      - image: gasbugs/http-go:v3
        imagePullPolicy: IfNotPresent
        name: http-go
```

### Replicaset 확인

새롭게 `http-go-855b9bcff4`이 추가된 것을 확인할 수 있음
```
$ kubectl get rs
NAME                 DESIRED   CURRENT   READY   AGE
http-go-6c4d9d5989   0         0         0       18m
http-go-855b9bcff4   3         3         3       84s
http-go-ccb794f48    0         0         0       70m
```

### Pod 확인

새로 생성된 Pod들 잘 돌아감
```
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
bash                       1/1     Running   0          27m
http-go-855b9bcff4-t29cb   1/1     Running   0          79s
http-go-855b9bcff4-tqcfd   1/1     Running   0          114s
http-go-855b9bcff4-tvkxc   1/1     Running   0          96s
```

### 모니터링 확인

v3로 잘 변경되고 무중단으로 애플리케이션 실행 중
```
Welcome! v3
Welcome! v3
Welcome! v3
Welcome! v3
Welcome! v3
Welcome! v3
Welcome! v3
```

### history 확인

REVISION 번호 3이 추가된 것 확인
```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deploy-v1.yaml --record=true
2         kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
3         kubectl edit deploy http-go --record=true
```


# 롤링 업데이트와 롤백 실습 - 4(롤백)

현재 v3로 애플리케이션이 돌아가고 있는데, `undo`를 사용하여 롤백

### 롤백 실행

```
$ kubectl rollout undo deploy http-go
deployment.apps/http-go rolled back
```

### 히스토리 확인

REVISION 번호 2는 제거되고 4가 추가된 것 확인  => undo를 실행하면서 4번이 최근것이 되었다는 의미

```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deploy-v1.yaml --record=true
3         kubectl edit deploy http-go --record=true
4         kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
```

### 모니터링 확인

애플리케이션이 롤백되어 v2로 돌아가는 것 확인
```
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
Welcome! v2
```

# 롤링 업데이트와 롤백 실습 - 5(특정 버전으로 롤백)

### 특정 버전으로 롤백

`--to-revision={REVISION 버전 번호}` 옵션 사용

```
$ kubectl rollout undo deploy http-go --to-revision=1
deployment.apps/http-go rolled back
```

### 히스토리 확인

REVISION 번호 1은 제거되고 5가 추가된 것 확인  
=> undo를 실행하면서 5번이 최근것이 되었다는 의미
```
$ kubectl rollout history deploy http-go
deployment.apps/http-go
REVISION  CHANGE-CAUSE
3         kubectl edit deploy http-go --record=true
4         kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
5         kubectl create --filename=http-go-deploy-v1.yaml --record=true
```

### 모니터링 확인

애플리케이션이 롤백되어 v1로 돌아가는 것 확인
```
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
Welcome! v1
```

# 연습 문제

- nginx:1.18 이미지를 사용하여 deployment 생성
  - Replicas: 10
  - maxSurge: 50%
  - maxUnavailable: 50%
- alpine:1.19 롤링 업데이트 수행
- nginx:1.18 롤백 수행

### Tip!

`--dry-run=client` 옵션을 사용하면 실제로 생성하지 않고 **문법이 맞는지 확인**해주는 꿀 기능이 있음
```
$ kubectl run --image alpine:3.4 alpine-deploy --dry-run=client
pod/alpine-deploy created (dry run)
```

### nginx-deploy-v1.yaml 생성

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    run: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 50%     
      maxUnavailable: 50%
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx-deployment
    spec:
      containers:
      - name: nginx-deployment
        image: nginx:1.18
        ports:
        - containerPort: 80

```

### 실행

```
$ kubectl create -f nginx-deploy-v1.yaml --record=true
deployment.apps/nginx-deployment created
```

### 히스토리 확인

```
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         kubectl create --filename=nginx-deploy-v1.yaml --record=true
```

### image 업데이트

`edit` 명령어로 업데이트 수행
```
$ kubectl edit deploy nginx-deployment --record=true
deployment.apps/nginx-deployment edited
```

이미지 버전을 1.18 => 1.19로 업데이트
```
    spec:
      containers:
      - image: nginx:1.19
        imagePullPolicy: IfNotPresent
        name: nginx-deployment
```

### Pod, Replicas 업데이트 확인
```
$ kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-56c949c7b-7bbfq   1/1     Running   0          72s
nginx-deployment-56c949c7b-qgkpb   1/1     Running   0          72s
nginx-deployment-56c949c7b-z9fw8   1/1     Running   0          72s

$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-56c949c7b    3         3         3       100s
nginx-deployment-78b6d5689f   0         0         0       5m10s
```

### 히스토리 확인
```
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         kubectl create --filename=nginx-deploy-v1.yaml --record=true
2         kubectl edit deploy nginx-deployment --record=true
```

### image 롤백 실행 및 히스토리 확인

롤백 실행
```
$ kubectl rollout undo deploy nginx-deployment --to-revision=1
deployment.apps/nginx-deployment rolled back
```

히스토리 확인
```
$ kubectl rollout history deploy nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         kubectl edit deploy nginx-deployment --record=true
3         kubectl create --filename=nginx-deploy-v1.yaml --record=true
```


