# 연습 문제

* 문제 1번

Namespace `blue`에 jenkins 이미지를 사용하는 `pod-jenkins` Deployment를 생성하고 이를 위한 Service `srv-jenkins`를 생성하라.

* 문제 2번

`default` Namespace의 http-go 이미지의 curl을 사용하여 `pod-jenkis`을 요청하라.
예시) `kubectl exec http-go-77cb5c879-29kld -- curl srv-jenkins.blue`

### 1번

* blue namespace 생성
```
$ kubectl create ns blue
namespace/blue created
```

* blue namespace 확인
```
$ kubectl get ns
NAME                  STATUS   AGE
blue                  Active   3s
default               Active   5d3h
```

* pod-jenkins Deployment YAML 작성

**pod-jenkins-deploy.yaml**
```
$ kubectl create deploy --image=jenkins pod-jenkins-deploy --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: pod-jenkins-deploy
  name: pod-jenkins-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-jenkins-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-jenkins-deploy
    spec:
      containers:
      - image: jenkins
        name: jenkins
        resources: {}
status: {}

$ kubectl create deploy --image=jenkins pod-jenkins-deploy --dry-run=client -o yaml > pod-jenkins-deploy.yaml
```

* srv-jenkins Service를 Deployment YAML에 추가(아래는 추가 전문)

**pod-jenkins-deploy.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: srv-jenkins
spec:
  selector:
    app: pod-jenkins-deploy
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
    app: pod-jenkins-deploy
  name: pod-jenkins-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-jenkins-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-jenkins-deploy
    spec:
      containers:
      - image: jenkins
        name: jenkins
        resources: {}
status: {}
```

* (blue namespace에 생성되도록) YAML 실행
```
$ kubectl create -f pod-jenkins-deploy.yaml -n blue
service/srv-jenkins created
deployment.apps/pod-jenkins-deploy created
```

* 생성된 쿠버네티스 리소스 확인
```
$ kubectl get all -n blue
NAME                                      READY   STATUS    RESTARTS   AGE
pod/pod-jenkins-deploy-6c8f5b65cb-b2b7z   1/1     Running   0          54s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/srv-jenkins   ClusterIP   10.102.107.10   <none>        80/TCP    54s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pod-jenkins-deploy   1/1     1            1           54s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/pod-jenkins-deploy-6c8f5b65cb   1         1         1       54s
```

### 2번

* http-go Deployment&Service YAML 작성

**http-go-deploy.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: srv-jenkins
spec:
  selector:
    app: pod-jenkins-deploy
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
    app: pod-jenkins-deploy
  name: pod-jenkins-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-jenkins-deploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-jenkins-deploy
    spec:
      containers:
      - image: jenkins
        name: jenkins
        resources: {}
status: {}
```

* (default namespace에 생성되도록) YAML 실행
```
$ kubectl create -f http-go-deploy.yaml
service/http-go-svc created
deployment.apps/http-go created
```

* 생성된 쿠버네티스 리소스 확인
```
$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/http-go-5c6f458dc9-m97w8   1/1     Running   0          11s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/http-go-svc   ClusterIP   10.111.217.56   <none>        80/TCP    11s
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   5d21h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/http-go   1/1     1            1           11s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-5c6f458dc9   1         1         1       11s
```

* default Namespace의 http-go 이미지의 curl을 사용하여 pod-jenkis을 요청 확인
```
$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
http-go-5c6f458dc9-m97w8   1/1     Running   0          42s

$ kubectl get svc -n blue
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
srv-jenkins   ClusterIP   10.102.107.10   <none>        80/TCP    4m58s

$ kubectl exec -it http-go-5c6f458dc9-m97w8 -- curl srv-jenkins.blue
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>


Authentication required
<!--
You are authenticated as: anonymous
Groups that you are in:

Permission you need to have (but didn't): hudson.model.Hudson.Administer
-->

</body></html>
```