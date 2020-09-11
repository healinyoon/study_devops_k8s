GKE에서 아래 예제 실행

# NodePort 예시 1

### NodePort Service 생성

* http-go-np.yaml 파일 작성
```
apiVersion: v1
kind: Service
metadata:
  name: http-go-np
spec:
  type: NodePort
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001
```

* Service 실행
```
$ kubectl create -f http-go-np.yaml
service/http-go-np created
```

* 생성된 NodePortIP 확인
```
$ kubectl get svc
NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
http-go-np    NodePort    10.4.11.238   <none>        80:30001/TCP   80s    <-- NodePort 타입이 생성된 것 확인
http-go-svc   ClusterIP   10.4.9.86     <none>        80/TCP         6h26m
kubernetes    ClusterIP   10.4.0.1      <none>        443/TCP        7h42m
```

### 방화벽 Open

* 프로젝트 setting
```
$ gcloud config set project healin-gke-service-20200910
Updated property [core/project].
```

* 방화벽 정책 등록
```
$ gcloud compute firewall-rules create http-go-svc-rule --
allow=tcp:30001
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/healin-gke-service-20200910/global/
firewalls/http-go-svc-rule].
Creating firewall...done.
NAME              NETWORK  DIRECTION  PRIORITY  ALLOW      DENY  DISABLED
http-go-svc-rule  default  INGRESS    1000      tcp:30001        False
```

### 외부에서 접근 확인

* Node 외부 IP 확인
```
$ kubectl get nodes -o wide
NAME                                     STATUS   ROLES    AGE     VERSION          INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cluster-default-pool-8fe2b9ad-14x1   Ready    <none>   7h43m   v1.15.12-gke.2   10.178.0.2    34.64.133.165   Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-cluster-default-pool-8fe2b9ad-6cff   Ready    <none>   7h43m   v1.15.12-gke.2   10.178.0.4    34.64.75.182    Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-cluster-default-pool-8fe2b9ad-hxpj   Ready    <none>   7h43m   v1.15.12-gke.2   10.178.0.3    34.64.199.253   Container-Optimized OS from Google   4.19.112+        docker://19.3.1
```

* 외부 접근 테스트
클러스터의 모든 Node(여기서는 3개)에서 동일한 Port로 서비스에 접근 가능한 것을 볼 수 있다.
```
$ curl 34.64.133.165:30001
Welcome! http-go-5d976c85c6-gnc8m

$ curl 34.64.75.182:30001
Welcome! http-go-5d976c85c6-v7jjs

$ curl 34.64.199.253:30001
Welcome! http-go-5d976c85c6-z7w65
```

# NodePort 예시 2

* tomcat을 NodePort로 서비스하기
* 30002 Port 사용

### NodePort Service 생성

* tomcat-deploy.yaml 파일 생성
```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-np
spec:
  type: NodePort
  selector:
    app: tomcat
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30002
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: tomcat
  name: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: tomcat
    spec:
      containers:
      - image: tomcat
        name: tomcat
        ports:
        - containerPort: 8080
        resources: {}
status: {}
```

* Service 실행
```
$ kubectl create -f tomcat-deploy.yaml
service/tomcat-np created
deployment.apps/tomcat created
```

* 생성된 NodePortIP 확인
```
$ kubectl get svc
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
http-go-lb    LoadBalancer   10.4.3.114    34.64.184.155   80:32747/TCP   29m
http-go-np    NodePort       10.4.11.238   <none>          80:30001/TCP   51m
http-go-svc   ClusterIP      10.4.9.86     <none>          80/TCP         7h16m
kubernetes    ClusterIP      10.4.0.1      <none>          443/TCP        8h
tomcat-np     NodePort       10.4.2.56     <none>          80:30002/TCP   2m3s
```

### 방화벽 Open

* 방화벽 정책 등록
```
$ gcloud compute firewall-rules create tmocat-svc-rule --a
llow=tcp:30002
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/healin-gke-service-20200910/global/
firewalls/tmocat-svc-rule].
Creating firewall...done.
NAME             NETWORK  DIRECTION  PRIORITY  ALLOW      DENY  DISABLED
tmocat-svc-rule  default  INGRESS    1000      tcp:30002        False
```

### 외부에서 접근 확인

* Node 외부 IP 확인
```
$ kubectl get nodes -o wide
NAME                                     STATUS   ROLES    AGE   VERSION          INTERNAL-IP   EXTERNAL-IP     
OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cluster-default-pool-8fe2b9ad-14x1   Ready    <none>   8h    v1.15.12-gke.2   10.178.0.2    34.64.133.165   
Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-cluster-default-pool-8fe2b9ad-6cff   Ready    <none>   8h    v1.15.12-gke.2   10.178.0.4    34.64.75.182    
Container-Optimized OS from Google   4.19.112+        docker://19.3.1
gke-cluster-default-pool-8fe2b9ad-hxpj   Ready    <none>   8h    v1.15.12-gke.2   10.178.0.3    34.64.199.253   
Container-Optimized OS from Google   4.19.112+        docker://19.3.1
```

* 외부 접근 테스트
```
$ curl 34.64.133.165:30002
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font
-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 
{font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#
525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> S
tatus Report</p><p><b>Description</b> The origin server did not find a current representation for the target res
ource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.37</h3></body><
/html>
```

# LoadBalancer 예시 2

* tomcat을 LoadBalance로 서비스하기

### LoadBalancer Service 생성

* tomcat-lb.yaml 파일 작성
```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-lb
spec:
  type: LoadBalancer
  selector:
    app: tomcat
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

* Service 실행
```
$ kubectl create -f tomcat-lb.yaml
service/tomcat-lb created
```

* 생성된 NodePortIP 확인(완료)
```
$ kubectl get svc -w
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
http-go-lb    LoadBalancer   10.4.3.114    34.64.184.155   80:32747/TCP   41m
http-go-np    NodePort       10.4.11.238   <none>          80:30001/TCP   63m
http-go-svc   ClusterIP      10.4.9.86     <none>          80/TCP         7h28m
kubernetes    ClusterIP      10.4.0.1      <none>          443/TCP        8h
tomcat-lb     LoadBalancer   10.4.14.120   34.64.68.204    80:32238/TCP   3m49s
tomcat-np     NodePort       10.4.2.56     <none>          80:30002/TCP   14m
```

### 외부에서 접근 확인

34.64.68.204 접속 확인
![](/images/10-Service2-NodePort-LoadBalancer-k8s-loadbalancer-web.png)

# LoadBalancer 예시 2

### LoadBalancer Service 생성

* http-go-lb.yaml 파일 작성
```
apiVersion: v1
kind: Service
metadata:
  name: http-go-lb
spec:
  type: LoadBalancer
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

* Service 실행
```
$ kubectl create -f http-go-lb.yaml
service/http-go-lb created
```

* 생성된 NodePortIP 확인(생성중)
```
$ kubectl get svc -w
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
http-go-lb    LoadBalancer   10.4.3.114    <pending>     80:32747/TCP   13s
http-go-np    NodePort       10.4.11.238   <none>        80:30001/TCP   21m
http-go-svc   ClusterIP      10.4.9.86     <none>        80/TCP         6h47m
kubernetes    ClusterIP      10.4.0.1      <none>        443/TCP        8h
```

* 생성된 NodePortIP 확인(완료)
```
$ kubectl get svc -w
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
http-go-lb    LoadBalancer   10.4.3.114    34.64.184.155   80:32747/TCP   2m12s
http-go-np    NodePort       10.4.11.238   <none>          80:30001/TCP   23m
http-go-svc   ClusterIP      10.4.9.86     <none>          80/TCP         6h49m
kubernetes    ClusterIP      10.4.0.1      <none>          443/TCP        8h
```

### 외부에서 접근 확인

* 34.63.184.155 접속 확인
![](/images/10-Service2-NodePort-LoadBalancer-k8s-loadbalancer-web2.png)

tomcat 버전 이슈로 404가 뜨는 거니까 당황하지 말고 tomcat 이미지를 변경해주면 된다.
```
$ kubectl edit deploy tomcat
deployment.extensions/tomcat edited

아래 내용 수정
image: consol/tomcat-7.0
```

* 다시 확인
![](/images/10-Service2-NodePort-LoadBalancer-k8s-loadbalancer-web3.png)