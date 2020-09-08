GKE에서 아래 예제 실행

# ClusterIP와 SessionAffinity 예제

### Step 1. Deployment와 Service 생성

* `--dry-run=client` 명령어로 문법 확인
```
$ kubectl create deploy --image=healinyoon/http-go http-go --dry-run=client -o yaml
```

아래 내용이 출력됨
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: http-go
    spec:
      containers:
      - image: healinyoon/http-go
        name: http-go
        resources: {}
status: {}
```

* Deployment yaml 파일 생성
```
$ kubectl create deploy --image=healinyoon/http-go http-go --dry-run=client -o yaml > http-go-deploy.yaml
```

* Deploy yaml 파일에 Service 추가
```
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  selector:
    run: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

---

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: http-go
    spec:
      containers:
      - image: healinyoon/http-go
        name: http-go
        resources: {}
status: {}
```

* Deployment 실행
```
$ kubectl create -f http-go-deploy.yaml
service/http-go-svc created
deployment.apps/http-go created
```

* 생성된 자원 확인
```
$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/http-go-65f7df44b9-mfbw4   1/1     Running   0          34s

NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/http-go-svc   ClusterIP   10.36.5.62   <none>        80/TCP    34s
service/kubernetes    ClusterIP   10.36.0.1    <none>        443/TCP   16m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/http-go   1/1     1            1           34s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-65f7df44b9   1         1         1       34s
```

* 할당받은 IP 확인
```
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP          NODE                         
              NOMINATED NODE   READINESS GATES
http-go-65f7df44b9-mfbw4   1/1     Running   0          101s   10.32.2.3   gke-cluster-1-default-pool-e2
45948b-p611   <none>           <none>
```

### Step 1. Deployment와 Service 생성
