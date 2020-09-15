# 목표

`ns-aravom` namespace를 생성하고, 해당 namespace에 외부에서 접속 가능한 nginx 생성

### 작업 순서

1) `ns-aravom` namespace 생성
2) nginx Deployment 및 NodePort Service yaml 파일 생성
3) `ns-aravom` namespace에 up

# 작업

### 1) `ns-aravom` namespace 생성

* namespace 생성
```
# kubectl create ns ns-aravom
namespace/ns-aravom created
```

* namespace 확인
```
# kubectl get ns
NAME                  STATUS   AGE
default               Active   52d
gitlab-managed-apps   Active   4d2h
kube-node-lease       Active   52d
kube-public           Active   52d
kube-system           Active   52d
ns-aravom             Active   15m
```

### 2) nginx Deployment 및 NodePort Service yaml 파일 생성

* Deployment yaml 문법 확인 및 `nginx-service-deploy.yaml` 파일 생성
```
# kubectl create deploy --image=nginx  nginx --dry-run=client -o yaml > nginx-service-deploy.yaml
```

* `nginx-service-deploy.yaml` 파일 Deployment 수정: spec에 ports 정보 추가
```
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
```

* `nginx-service-deploy.yaml` 파일 Service 추가
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-np
spec:
  type: NodePort           # NodePort type
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80              # Service Port
      targetPort: 80        # Pod Port
      nodePort: 31020       # 최종적으로 서비스되는 Port

---
```

* 최종 `nginx-service-deploy.yaml` 파일
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-np
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 31020

---

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
: {}
```

### 3) `ns-aravom` namespace에 up

* `ns-aravom` namespace에 `nginx-service-deploy.yaml` 파일 실행
```
# kubectl create -f nginx-service-deploy.yaml -n ns-aravom
```

* 확인
  * 실제로 Pod가 떠있는 Node는 `gs-gpu-1080ti-07(10.231.238.47)` 서버  
  * 31020 Port 사용
  * 현재 replicaSet은 1개(scaling 가능, 10개까지 테스트 완료)

```
# kubectl get all -n ns-aravom -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
pod/nginx-d46f5678b-crhvn   1/1     Running   0          26m   10.38.0.1   gs-gpu-1080ti-07   <none>           <none>

NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   SELECTOR
service/nginx-svc-np   NodePort   10.103.56.40   <none>        80:31020/TCP   26m   app=nginx

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx   1/1     1            1           26m   nginx        nginx    app=nginx

NAME                              DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES   SELECTOR
replicaset.apps/nginx-d46f5678b   1         1         1       26m   nginx        nginx    app=nginx,pod-template-hash=d46f5678b
```

* 외부 접속 확인
모든 master, Worker IP로 동일한 Pod에 접근 되는 것 확인
(단 외부 방화벽이 해제되지 않은 10.231.238.45-48 대역 IP로는 외부 접속 불가)
![](/k8s-core-concepts/images/08-Namespaces-1.png)