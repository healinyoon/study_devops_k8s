# Replication Controller

### Replication 생성

```
$ kubectl create -f http-go-rc.yaml 
replicationcontroller/http-go created
```

### 생성된 Replication 확인

```
$ kubectl get rc
NAME      DESIRED   CURRENT   READY   AGE
http-go   3         3         0       6s
```

### Pod 확인

```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-2dqcs   1/1     Running   0          62s
http-go-jl7gp   1/1     Running   0          62s
http-go-nk5jl   1/1     Running   0          62s
```

### Replication Controller에 의한 Pod 자동 복구 확인

임의로 pod 제거
```
$ kubectl delete pod http-go-2dqcs
pod "http-go-2dqcs" deleted
```

Pod 자동 복구 확인
```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-76jhc   1/1     Running   0          2m3s
http-go-jl7gp   1/1     Running   0          4m55s
http-go-nk5jl   1/1     Running   0          4m55s
```

### Replication Controller이 Label로 Pod를 매칭하는 원리 이해하기
> Label 제거시 Replication Controller에 의한 변동 사항을 확인

현재 Pod의 Label 확인
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          3m9s    app=http-go
http-go-jl7gp   1/1     Running   0          6m1s    app=http-go
http-go-nk5jl   1/1     Running   0          6m1s    app=http-go
http-go-v2      1/1     Running   0          3h43m   creation_method=manual-v2
```

`http-go-76jhc` Pod의 Label 제거
```
$ kubectl label pod http-go-76jhc app-
pod/http-go-76jhc labeled 
```

Label이 제거 되어 해당 Pod가 더이상 Replication Controller에 의해 관리되지 않게 됨 => 제거되었다고 생각하고 새로 생성 해줌
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          4m31s   <none>                     # Label이 제거되어 관리되지 않는 Pod가 됨
http-go-gdl6h   1/1     Running   0          30s     app=http-go                # Replication Controller에 의해 관리되는 새로 생성된 Pod
http-go-jl7gp   1/1     Running   0          7m23s   app=http-go
http-go-nk5jl   1/1     Running   0          7m23s   app=http-go
http-go-v2      1/1     Running   0          3h44m   creation_method=manual-v2
```

### rc의 Pod들이 어느 노드에 있는지 확인

```
$ kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP          NODE           NOMINATED NODE   READINESS GATES
http-go-gdl6h   1/1     Running   0          5m21s   10.32.0.5   k8s-worker02   <none>           <none>
http-go-jl7gp   1/1     Running   0          12m     10.40.0.3   k8s-worker01   <none>           <none>
http-go-nk5jl   1/1     Running   0          12m     10.40.0.1   k8s-worker01   <none>           <none>
```

### 노드가 강제로 연결이 끊길 경우 관찰

```
$ kubectl get nodes -w  # 이렇게 해놓고, 네트워크를 끊어보고 관찰
```

### Replication Controller 정보 확인

```
$ kubectl get rc http-go -o wide
NAME      DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES               SELECTOR
http-go   3         3         3       20m   http-go      healinyoon/http-go   app=http-go
```

### Replication Controller 삭제

```
$ kubectl delete rc http-go
replicationcontroller "http-go" deleted
```
 
### Replication Controller 스케일링

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

# Replica Set

### Replica Set 생성
```
$ kubectl create -f frontend.yaml
replicaset.apps/frontend created
```

### 생성된 Replica Set 확인
```
$ kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       4m53s
```

### Pod 확인
```
$ kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
frontend-9lrkc   1/1     Running   0          4m23s   tier=frontend
frontend-s9qn4   1/1     Running   0          4m23s   tier=frontend
frontend-sspqg   1/1     Running   0          4m23s   tier=frontend
```

### rs 상세 조회
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

### rs 삭제
```
$ kubectl delete rs frontend
replicaset.apps "frontend" deleted
```