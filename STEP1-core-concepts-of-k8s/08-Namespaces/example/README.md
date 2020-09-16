# namespace 생성

### 방법 1)
```
$ kubectl create ns office
namespace/office created
```

### 방법 2)
```
$ kubectl create ns office --dry-run=client -o yaml > office-ns.yaml
$ kubectl create -f office-ns.yaml
```

# namespace에 자원 할당

### nginx 자원 할당
```
$ kubectl create deploy nginx --image nginx -n office
deployment.apps/nginx created
```

### 할당된 자원 조회
```
$ kubectl get all -n office
NAME                         READY   STATUS              RESTARTS   AGE
pod/nginx-6799fc88d8-gtl85   0/1     ContainerCreating   0          4s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   0/1     1            0           4s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   1         1         0       4s
```

# 모든 namespace 내용 조회

### 조회
```
$ kubectl get all --all-namespaces
```

아래 내용이 출력됨
```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-f9fd979d6-9vcgn                1/1     Running   0          3d21h
kube-system   pod/coredns-f9fd979d6-mczl5                1/1     Running   0          3d21h
kube-system   pod/etcd-k8s-master01                      1/1     Running   0          3d21h
kube-system   pod/kube-apiserver-k8s-master01            1/1     Running   0          3d21h
kube-system   pod/kube-controller-manager-k8s-master01   1/1     Running   0          3d21h
kube-system   pod/kube-proxy-4g924                       1/1     Running   0          3d21h
kube-system   pod/kube-proxy-647b5                       1/1     Running   0          3d21h
kube-system   pod/kube-proxy-lzzlm                       1/1     Running   0          3d21h
kube-system   pod/kube-scheduler-k8s-master01            1/1     Running   0          3d21h
kube-system   pod/weave-net-gn4sw                        2/2     Running   0          3d21h
kube-system   pod/weave-net-hgj79                        2/2     Running   0          3d21h
kube-system   pod/weave-net-hj2gf                        2/2     Running   0          3d21h
office        pod/nginx-6799fc88d8-gtl85                 1/1     Running   0          99s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  5m20s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3d21h

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   3d21h
kube-system   daemonset.apps/weave-net    3         3         3       3            3           <none>                   3d21h

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           3d21h
office        deployment.apps/nginx     1/1     1            1           99s

NAMESPACE     NAME                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-f9fd979d6   2         2         2       3d21h
office        replicaset.apps/nginx-6799fc88d8    1         1         1       99s
```

# default namespace 변경하기(실패)

사용자 설정 파일 변경
```
$ vi ~/.kube/config

아래 내용 수정
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
    namespace: office           <-- 이 부분 추가
  name: kubernetes-admin@kubernetes
```

# namespace 제거

```
$ kubectl delete ns office
namespace "office" deleted
```

# 연습문제

### Q1. 현재 시스템에는 몇 개의 Namespace 존재하는가? 4개
```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   3d21h
kube-node-lease   Active   3d21h
kube-public       Active   3d21h
kube-system       Active   3d21h
```

### Q2. kube-system에는 몇 개의 pod가 존재하는가? 13개
```
$ kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-9vcgn                1/1     Running   0          3d21h
coredns-f9fd979d6-mczl5                1/1     Running   0          3d21h
etcd-k8s-master01                      1/1     Running   0          3d21h
kube-apiserver-k8s-master01            1/1     Running   0          3d21h
kube-controller-manager-k8s-master01   1/1     Running   0          3d21h
kube-proxy-4g924                       1/1     Running   0          3d21h
kube-proxy-647b5                       1/1     Running   0          3d21h
kube-proxy-lzzlm                       1/1     Running   0          3d21h
kube-scheduler-k8s-master01            1/1     Running   0          3d21h
weave-net-gn4sw                        2/2     Running   0          3d21h
weave-net-hgj79                        2/2     Running   0          3d21h
weave-net-hj2gf                        2/2     Running   0          3d21h
```

### Q3. ns-jenkins namespace를 생성하고, jenkins pod를 배치해라

1) k8s docs에서 pod 템플릿 복사
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
    image: busybox
    command: ['sh', '-c', 'echo 안녕하세요 쿠버네티스! && sleep 3600']
```

2) jenkins-ns.yaml 파일 생성
```
$ kubectl create ns ns-jenkins --dry-run=client -o yaml > jenkins-ns.yaml
```

3) jenkins-ns.yaml 파일 수정 및 Pod 추가
```
apiVersion: v1
kind: Namespace
metadata:
  name: ns-jenkins

---      <-- !여러 개의 yaml 파일을 나열할 때 구분하기 위해 넣어주는 것!

apiVersion: v1
kind: Pod
metadata:
  name: jenkins
  namespace: ns-jenkins
spec:
  containers:
  - name: jenkins
    image: jenkins
    ports:
    - containerPort: 8080
```

4) yaml 파일 실행
namespace와 pod가 순차적으로 생성되는 것을 확인할 수 있음
```
$ kubectl create -f jenkins-ns.yaml
namespace/ns-jenkins created
pod/jenkins created
```

5) 생성된 Pod 확인
```
$ kubectl get pod -n ns-jenkins
NAME      READY   STATUS    RESTARTS   AGE
jenkins   1/1     Running   0          65s
```

### Q4. coredns는 어느 namespace가 존재하는가? kube-system

관리자 입장에서 조회하는 것
```
$ kubectl get pod --all-namespaces | grep coredns
kube-system   coredns-f9fd979d6-9vcgn                1/1     Running   0          4d10h
kube-system   coredns-f9fd979d6-mczl5                1/1     Running   0          4d10h
```