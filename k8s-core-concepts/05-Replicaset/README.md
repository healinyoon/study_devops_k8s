# 레플리케이션 컨트롤러

- 구 버전(v1.8 이전)의 쿠버네티스에서 사용
- pod가 항상 실행되도록 유지하는 쿠버네티스 리소스
  - 노드가 클러스터에서 사라지는 경우 해당 pod를 감지하고 대체 pod 생성
  - 실행 중인 pod 목록 지속적 모니터링, 실제 pod 수가 원하는 수와 항상 일치하는 지 확인

## 레플리케이션 컨트롤러의 세 가지 요소 

1) 레이블 셀렉터: 레플리케이션컨트롤러가 관리하는 포드 범위를 결정(rc는 label로 pod를 관리하기 때문)
2) 복제본 수: 실행해야하는 포드 수 결정
3) 포드 템플릿: 새로운 포드의 모양을 설명

## 레플리케이션 컨트롤러의 장점

- 포드가 없는 경우 새 포드를 항상 실행
- 노드에 장애 발생 시 다른 노드에 복제본 생성
- 수동, 자동으로 수평 스케일링

## 레플리케이션 컨트롤러 실습

#### 레플리케이션 생성
```
$ kubectl create -f http-go-rc.yaml 
replicationcontroller/http-go created
```

#### 레플리케이션 확인
```
$ kubectl get rc
NAME      DESIRED   CURRENT   READY   AGE
http-go   3         3         0       6s
```

#### pod 확인
```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-2dqcs   1/1     Running   0          62s
http-go-jl7gp   1/1     Running   0          62s
http-go-nk5jl   1/1     Running   0          62s
```

#### 자동 복구 확인

임의로 pod 제거
```
$ kubectl delete pod http-go-2dqcs
pod "http-go-2dqcs" deleted
```

자동으로 복구 되는 것 확인
```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-76jhc   1/1     Running   0          2m3s
http-go-jl7gp   1/1     Running   0          4m55s
http-go-nk5jl   1/1     Running   0          4m55s
```

#### Label 확인
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          3m9s    app=http-go
http-go-jl7gp   1/1     Running   0          6m1s    app=http-go
http-go-nk5jl   1/1     Running   0          6m1s    app=http-go
http-go-v2      1/1     Running   0          3h43m   creation_method=manual-v2
```

#### Label  제거시 변동 사항 확인

Label 제거
```
$ kubectl label pod http-go-76jhc app-
pod/http-go-76jhc labeled 
```

Label이 제거 되어 관리되지 않게 됨 => 제거되었다고 생각하고 새로 복구 해줌
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          4m31s   <none>
http-go-gdl6h   1/1     Running   0          30s     app=http-go
http-go-jl7gp   1/1     Running   0          7m23s   app=http-go
http-go-nk5jl   1/1     Running   0          7m23s   app=http-go
http-go-v2      1/1     Running   0          3h44m   creation_method=manual-v2
```

#### rc가 어느 노드에 있는지 확인
```
$ kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP          NODE           NOMINATED NODE   READINESS GATES
http-go-gdl6h   1/1     Running   0          5m21s   10.32.0.5   k8s-worker02   <none>           <none>
http-go-jl7gp   1/1     Running   0          12m     10.40.0.3   k8s-worker01   <none>           <none>
http-go-nk5jl   1/1     Running   0          12m     10.40.0.1   k8s-worker01   <none>           <none>
```

#### 노드가 강제로 연결이 끊길 경우 관찰
```
$ kubectl get nodes -w  # 이렇게 해놓고, 네트워크를 끊어보고 관찰
```

#### 레플레케이션 컨트롤러 정보 확인
```
$ kubectl get rc http-go -o wide
NAME      DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES               SELECTOR
http-go   3         3         3       20m   http-go      healinyoon/http-go   app=http-go
```

#### 레플리케이션 컨트롤러 삭제
```
$ kubectl delete rc http-go
replicationcontroller "http-go" deleted
```
 
#### 레플리케이션 컨트롤러 스케일링

방법 1)
```
$ kubectl scale rc http-go --replicas=5
replicationcontroller/http-go scaled

$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-4xpzt   1/1     Running   0          2m
http-go-6g9jh   1/1     Running   0          2m
http-go-wqjmq   1/1     Running   0          81s
http-go-x7d92   1/1     Running   0          2m
http-go-xjpt8   1/1     Running   0          81s
```

방법 2)
```
$ kubectl edit rc http-go
vim으로 파일이 열림. spec 하위의 replicas 수정

$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-4xpzt   1/1     Running   0          4m58s
http-go-x7d92   1/1     Running   0          4m58s
```

방법 3)
```
$ cp http-go-rc.yaml http-go-rc-v2.yaml
$ vi http-go-rc-v2.yaml
vim으로 파일이 열림. spec 하위의 replicas 수정

$ kubectl apply -f http-go-rc-v2.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
replicationcontroller/http-go configured

$ kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
http-go-4xpzt   1/1     Running   0          8m43s
http-go-99lzc   1/1     Running   0          68s
http-go-qkbhc   1/1     Running   0          68s
http-go-sc8c9   1/1     Running   0          68s
http-go-x7d92   1/1     Running   0          8m43s
```

# 레플리카셋

- 쿠버네티스 1.8 버전부터 Deployment, Daemonset, ReplicaSet, SatetfulSet API가 베타로 업데이트되고, 1.9 버전부터 정식으로 업데이트 됨
- 레플리카셋은 레플리케이션컨트롤러를 완전히 대체 가능
- 일반적으로 레플리카셋을 직접 생성하지 않고 상위 수준의 디플로이먼트 리소스를 만들 때 자동으로 생성

## 레플리케이션 컨트롤러 vs 레플리카셋

- 거의 동일하게 동작
- 레플리카셋이 더 풍부한 pod selector 사용 가능
  - 레플리케이션 컨트롤러: 특정 label의 key, value가 일치하는 pod 매치
  - 레플리카셋: 유연한  label을 매칭하는 matchExpressions 사용하여 pod 매치

## 레플리카셋 실습

#### 레플리카셋 생성
```
$ kubectl create -f frontend.yaml
replicaset.apps/frontend created
```

#### 레플리카셋 확인
```
$ kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       4m53s
```

#### pod 확인
```
$ kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
frontend-9lrkc   1/1     Running   0          4m23s   tier=frontend
frontend-s9qn4   1/1     Running   0          4m23s   tier=frontend
frontend-sspqg   1/1     Running   0          4m23s   tier=frontend
```

#### rs 상세 조회
```
$ kubectl describe rs frontend
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  6m25s  replicaset-controller  Created pod: frontend-s9qn4
  Normal  SuccessfulCreate  6m25s  replicaset-controller  Created pod: frontend-9lrkc
  Normal  SuccessfulCreate  6m25s  replicaset-controller  Created pod: frontend-sspqg
```

#### rs 삭제
```
$ kubectl delete rs frontend
replicaset.apps "frontend" deleted
```