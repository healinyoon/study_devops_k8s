# 이슈
특정 Node에 Pod가 집중되어 배포되는 현상 발생

# 목적
Pod의 리소스 점유량 최소/최대 값을 지정하여, 각 Worker Node마다 Pod가 고르게 배포되게 한다.

# 알아야 할 점
리소스 사용량과 리소스 점유량을 구분한다.
* 사용량: Pod가 실제로 사용하고 있는 리소스
* 점유량: Pod가 점유하고 있다고 여겨지는 리소스 => **Pod 배포시 리소스 점유량을 기준으로 배포한다.**

# 내용

### 목표
리소스 점유량 `최소/최대` 값에 대한 제어하여 pod를 배포하기 => 의도한 대로 각 Worker Node마다 `Pod가 고르게 배포`되는 것을 확인하자
 
### yaml 설정
* request: 각 Pod가 점유하는 최소 리소스 = 이 만큼은 무조건 가지고 있어야 함
* limit: 각 Pod가 점유 가능한 최대 리소스 = 이 만큼 까지만 사용 가능함(But 이미 점유된 리소스를 뺏어올 순 없음)

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: http-go
  labels:
    app: http-go
spec:
  replicas: 6
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
      containers:
      - name: http-go
        image: healinyoon/http-go
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "3153476Ki" <= 노드 memory의 1/3
            cpu: "700m"         <= 노드 cpu의 1/3
          limits:
            memory: "4153476Ki" <= 노드 memory의 1/2
            cpu: "1000m"        <= 노드 cpu의 1/2
```

* replica = 6 하였을때 pod는 아래와 같이 배포된다.
```
$ kubectl get all -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
pod/http-go-bvw9h   1/1     Running   0          11s   10.32.0.3   worker2   <none>           <none>
pod/http-go-lf5tc   1/1     Running   0          11s   10.40.0.3   worker1   <none>           <none>
pod/http-go-p2knq   1/1     Running   0          11s   10.40.0.2   worker1   <none>           <none>
pod/http-go-qcvf9   1/1     Running   0          11s   10.32.0.2   worker2   <none>           <none>
pod/http-go-qhz2h   1/1     Running   0          11s   10.38.0.2   master    <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15d   <none>

NAME                      DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES               SELECTOR
replicaset.apps/http-go   6         6         5       11s   http-go      healinyoon/http-go   app=http-go
```

master node의 리소스가 부족한 경우 Pending이되고, worker1과 worker2도 pod가 requests를 점유하고 있기에 해당 node로 배포되지 않음
(여기서 리소스가 부족함은 사용률이 아닌 점유율을 의미한다.) 
```
pod/http-go-4ccrk   0/1     Pending   0          11s   <none>      <none>    <none>           <none>
```

* Node별 리소스 사용량 체크
```
$ kubectl top nodes
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master    440m         22%    4548Mi          58%
worker1   83m          4%     2445Mi          31%
worker2   54m          2%     519Mi           6%
```

실제 사용되는 리소스의 %률을 확인해보니 master와 worker 그 어디에도 pending된 pods가 배포되지 않음으로, `requests`옵션이 제대로 사용되고 있음을 확인할 수 있다.