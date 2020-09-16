GKE에서 아래 예제 실행

# ClusterIP와 SessionAffinity 예제

### Step 1. Deployment와 Service 생성하기

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

하나의 yaml 파일에 `---`을 사용해서 여러가지 자원을 동시에 실행시킬 수 있다.

```
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  selector:
    app: http-go
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
        ports:
        - containerPort: 8080
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
pod/http-go-5d976c85c6-v7jjs   1/1     Running   0          140m

NAME                  TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/http-go-svc   ClusterIP   10.4.9.86    <none>        80/TCP    140m
service/kubernetes    ClusterIP   10.4.0.1     <none>        443/TCP   3h35m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/http-go   1/1     1            1           140m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-5d976c85c6   1         1         1       140m
```

### Step 2. 생성된 Service 정보 살펴보기

생성된 Service 정보를 살펴보고 동작 원리를 이해할 수 있다.

* 할당받은 IP 확인

Service를 통해 10.0.2.5 IP를 할당 받은 것을 확인할 수 있다.

```
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP         NODE                                     NOMINATED NODE   READINESS GATES
http-go-5d976c85c6-v7jjs   1/1     Running   0          141m   10.0.2.5   gke-cluster-default-pool-8fe2b9ad-6cff   <none>           <none>
```

* svc 조회

svc 조회를 통해서도 IP(Endpoint) 확인이 가능하다.

```
$ kubectl describe svc http-go-svc
Name:              http-go-svc
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=http-go
Type:              ClusterIP
IP:                10.4.9.86
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.0.2.5:8080
Session Affinity:  None
Events:            <none>
```

* Scale 변경 및 확인
```
$ kubectl scale deploy http-go --replicas=5
deployment.extensions/http-go scaled

$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP         NODE                                     NOMINATED NODE   READINESS GATES
http-go-5d976c85c6-gnc8m   1/1     Running   0          49m     10.0.2.6   gke-cluster-default-pool-8fe2b9ad-6cff   <none>           <none>
http-go-5d976c85c6-n5rlg   1/1     Running   0          49m     10.0.2.7   gke-cluster-default-pool-8fe2b9ad-6cff   <none>           <none>
http-go-5d976c85c6-v7jjs   1/1     Running   0          3h16m   10.0.2.5   gke-cluster-default-pool-8fe2b9ad-6cff   <none>           <none>
http-go-5d976c85c6-wpc6h   1/1     Running   0          49m     10.0.1.4   gke-cluster-default-pool-8fe2b9ad-hxpj   <none>           <none>
http-go-5d976c85c6-z7w65   1/1     Running   0          49m     10.0.0.9   gke-cluster-default-pool-8fe2b9ad-14x1   <none>           <none>
```

Scale을 변경하면 다음과 같이 Endpoints가 많아진 것을 확인할 수 있다.
```
$ kubectl describe svc http-go-svc
Name:              http-go-svc
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=http-go
Type:              ClusterIP
IP:                10.4.9.86
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.0.0.9:8080,10.0.1.4:8080,10.0.2.5:8080 + 2 more...
Session Affinity:  None
Events:            <none>
```

### Step 3. SessionAffinity=ClientIP로 변경해보기

SessionAffinity=Client로 변경하면 client가 계속 동일한 Pod로 접근 가능하도록 연결해줄 수 있다.

* http-go-svc 설정 변경
```
$ kubectl edit svc http-go-svc
service/http-go-svc edited
```

아래 내용 수정
```
sessionAffinity: ClientIP
```

변경된 Service의 ClusterIP 확인 
```
$ kubectl get svc http-go-svc
NAME          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
http-go-svc   ClusterIP   10.4.9.86    <none>        80/TCP    3h25m
```

* sessionAffinity=ClientIP 동작 확인

이제 sessionAffinity를 ClientIP로 변경해주었기 때문에, 동일한 Client가 접근시 동일한 Pod로 접속시켜준다(=로드밸런스 시키지 않는다).

이를 테스트 하기 위해 Client Pod를 띄우고 테스트 해보자
```
$ kubectl run -it --rm --image=busybox bash
If you don't see a command prompt, try pressing enter.
/ # wget -O- -q 10.4.9.86
Welcome! http-go-5d976c85c6-gnc8m
/ # wget -O- -q 10.4.9.86
Welcome! http-go-5d976c85c6-gnc8m
/ # wget -O- -q 10.4.9.86
Welcome! http-go-5d976c85c6-gnc8m
/ # wget -O- -q 10.4.9.86
Welcome! http-go-5d976c85c6-gnc8m
/ # wget -O- -q 10.4.9.86
Welcome! http-go-5d976c85c6-gnc8m
```

위와 같이 여러 차례 Client가 Service로 접근을 시도하면 동일한 Pod로 계속해서 연결해주는 것을 볼 수 있다.  

그렇다면 Client의 접속 정보는 얼마 동안 유지 될까?  
[Kubernetes docs](https://kubernetes.io/docs/concepts/services-networking/service/)에 따르면 기본 값은 10800초(3시간)이다.
