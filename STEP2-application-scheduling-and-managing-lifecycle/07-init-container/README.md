# Init Container

### 개요
* Pod container 실행 전에 초기화 역할을 하는 container이다.
* 완전히 초기화가 진행된 다음에 main container를 실행시킨다.
* Init container가 실패하면, 성공할 때까지 Pod를 반복해서 재시작 한다.
    * restartPolicydp Never를 설정하면 재시작하지 않는다.
    
![](/STEP2-application-scheduling-and-managing-lifecycle/images/07-init-containe1.png)

### Init Container 예시
아래의 YAML은 `mydb`와 `myservice`가 탐지될 때까지 Init container가 멈추기 않고 실행된다. 
이는 main container가 특정 service에 의존성을 가지는 경우 Init container를 사용하는 예시를 보여준다.

#### 1. Init container를 사용하는 Pod YAML 작성

> pod-myapp.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

#### 2. Pod 생성
```
$ kubectl create -f pod-myapp.yaml
pod/myapp-pod created

$ kubectl get pod -w
NAME                      READY   STATUS      RESTARTS   AGE
myapp-pod                 0/1     Init:0/2    0          7s
```

위의 출력을 읽는 방법: `STATUS`의 `Init:0/2`가 모두 완료된 후에 `READY`의 `0/1`이 채워지게 된다.
하지만 위의 Init container의 조건을 만족시키지 않았기 때문에 main container는 생성되지 않는다.


#### 3. myservice와 mydb YAML 작성 및 생성

위의 Init container 조건을 만족시키기 위해, myservice와 mydb를 생성한다.

> svc-myservice-mydb.yaml
```
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

```
$ kubectl create -f svc-myservice-mydb.yaml
service/myservice created
service/mydb created
```

#### 4. Main container 실행 확인

위에서 Init container 조건을 만족시켜주었으므로, Init container가 실행되고 이어서 main container도 생성되는 것을 확인할 수 있다.

```
$ kubectl get pod -w
NAME                      READY   STATUS      RESTARTS   AGE
myapp-pod                 0/1     Init:0/2    0          7s
myapp-pod                 0/1     Init:1/2    0          8m14s
myapp-pod                 0/1     PodInitializing   0          8m15s
myapp-pod                 1/1     Running           0          8m16s
```
