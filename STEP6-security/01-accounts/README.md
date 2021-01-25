# Accounts

### Accounts Type

Accounts Type에는 2가지 타입이 존재한다.

* 사용자를 위한 User Account(= Static Token File)
* 애플리케이션을 위한 Service Account

# Static Token File

### 개요

* Account를 만드는 가장 쉬운 방법이다.
* apiserver 서비스를 실행할 때, `--token-auth-file={filename}.csv`를 전달한다.
    * kube-apiserver의 StaticPod를 수정해야 한다. = apiserver를 다시 시작해야 적용된다.
* csv file은 패스워드(token), 아이디(사용자 명), 사용자 UID 정보를 저장한다. 그 외 추가적인 정보를 저장할 수 있다.

> (참고) fiel.csv
```
passwd01,usr01,uid01,"group1"
passwd02,usr02,uid02
passwd03,usr03,uid03
```

### 사용 방법

Static toekn file이 적용된 후에는 바로 사용 가능하며, 적용 후 사용 방법은 다음과 같다(Static token file 적용 방법은 아래의 연습 문제에서 자세히 다룬다).

#### Header에 추가하여 사용하는 방법
HTTP 요청을 진행할 때, 다음의 내용을 header에 포함해서 사용해야 한다.
```
Authorization: Bearer $TOKEN
```

```
$ TOKEN=passwd01
$ APISERVER=https://127.0.0.1:6443
$ curl -x GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

#### Kubectl에 등록하고 사용하는 방법
```
$ kubectl config set-credentials user01 --token=passwd01
$ kubectl config set-context --cluster=kubernetes --namespace=frontend --user=user01

$ kubectl get pod --user user01
```

`--cluster=kubernetes`, `--namespace=frontend`, `--user=user01` 옵션은 여러 개의 Kubernets cluster를 사용하는 경우 유용하다.

# Service Account

### 개요

Pod에 별도의 설정을 주지 않으면 기본적인 Service Account가 생성된다. `default` Service Account를 조회해보면 다음과 같은 형식을 가지고 있다.
```
$ kubectl get sa default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2020-09-08T07:35:43Z"
  name: default
  namespace: default
  resourceVersion: "343"
  uid: f1199efa-e61b-45ca-9d17-a6522d94e4e3
secrets:
- name: default-token-fvtzt
```

### Service Account 생성 및 사용 방법

추가적인 Service Account가 필요하다면 다음 명령어로 생성할 수 있다.
```
$ kubectl create sa sa-test
serviceaccount/sa-test created
``` 

어떤 Pod가 어떤 Service Account를 사용하게 할 것인가는 Pod YAML에 다음과 같이 매칭해주면 된다.
```
spec.serviceAccountName: {service account name(예: sa-test)}
```

# Accounts 연습 문제

### 1. Service Accounts
* http-go라는 이름을 가진 ServiceAccount를 생성한다.
* http-go Pod를 생성하고, http-go ServiceAccount를 사용하도록 설정한다.

#### 1.1. ServiceAccount 생성
```
$ kubectl create sa http-go
```

#### 1.2. 생성된 ServiceAccount 조회
```
$ kubectl get sa http-go -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2021-01-15T06:47:54Z"
  name: http-go
  namespace: default
  resourceVersion: "27355732"
  uid: 387a2acf-5a8e-469c-916e-560be94a5eee
secrets:
- name: http-go-token-dmjlf
```

#### 1.3. ServiceAccount를 사용하는 Pod YAML 작성

> http-go.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-go
  labels:
    app: http-go
spec:
  replicas: 1
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
        image: gasbugs/http-go
        ports:
        - containerPort: 80
      serviceAccountName: http-go
```

#### 1.4. Pod 생성
```
$ kubectl create -f http-go.yaml
```

### 2. Static Token File
* 다음 csv file을 생성하고, apiserver에 등록 및 apiserver를 재시작한다.
```
passwd01,user01,uid001,"group01"
passwd02,user02,uid002
passwd03,user03,uid003
passwd04,user04,uid004
```
* master node에 문제가 발생하는가? 어떤 서비스에 문제가 발생하는가?
* 문제가 발생하는 서비스의 contaienr를 확인하기 위해 `docker logs`를 사용하고 그 해결 방법을 찾아라.
* 정상적으로 서비스가 시작됐다면, kubectl에 user 정보를 등록하고, 등록한 user 권한으로 `kubectl get pod` 요청을 수행해라.
    * 반드시 Forbidden이 출력되어야 한다(user는 존재하되, `get pod`를 할 권한은 없다는 의미).


#### 2.1. csv 파일 작성

> user.csv
```
passwd01,user01,uid001,"group01"
passwd02,user02,uid002
passwd03,user03,uid003
passwd04,user04,uid004
```

#### 2.2. csv 파일을 kube-apiserver의 Volume Mounts된 경로로 복사
```
$ sudo cp user.csv /etc/kubernetes/pki/user.csv
```

File을 복사하는 이유는 아래의 2.3에서 알 수 있다.

#### 2.3. kube-apiserver의 StaticPod를 수정
```
$ sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml


① 옵션 추가
- --token-auth-file=/etc/kubernetes/pki/user.csv                        <-- Container 내부 경로임을 명심하자!

② file에 대한 Volume Mounts

여기서는 기존에 이미 mount된 /etc/kubernetes/pki 경로를 활용하였으므로, mount 된 상태이다.
```
 
file.csv 파일의 경로가 Container 내부에서 접근 가능한 경로가 아니면(=Volume Mount를 해주지 않으면) 다음과 같은 에러가 발생하니 주의하자!

```
Error: invalid authentication config: open {file 경로}/user.csv: no such file or directory
```

#### 2.4. kube-apiserver 재시작 확인
```
$ watch "sudo docker ps -a | grep api"

422ff860ad2a   ca9843d3b545                   "kube-apiserver --ad…"   13 seconds ago   Up 12 seconds
         k8s_kube-apiserver_kube-apiserver-master_kube-system_874bd2c53ace47a5ad1d7921ee8e6cb2_7
ffb85fd5df71   k8s.gcr.io/pause:3.2           "/pause"                 2 minutes ago    Up 2 minutes
         k8s_POD_kube-apiserver-master_kube-system_874bd2c53ace47a5ad1d7921ee8e6cb2_3
```

kube-apiserver container가 정상 동작할 때까지 기다린다.

#### 2.5. Kubectl에 등록하고 사용

아래의 명령어를 순차적으로 실행한다.

① 사용할 계정과 패스워드를 setting 한다.
```
$ kubectl config set-credentials user01 --token=passwd01
User "user01" set.
```

② 생성한 계정과 cluster 연결
```
$ kubectl config set-context user01-context --cluster=kubernetes --namespace=frontend --user=user01
Context "user01-context" created.
```

* `set-context`에서 context의 의미
    * ①에서 생성한 계정과 이 kubernetes 서버(cluster)를 연결하는 명령어
    * 이 kubernetes 서버(cluster)에 접속할 때는 이 user(user01)를 사용하겠다를 **user01-context**에 등록합니다.
* `--cluster=kubernetes`: kubernetes는 현재 우리가 사용하고 있는 cluster를 의미힌다.

③ 생성한 계정으로 로그인(admin -> user01 계정으로 switch)
```
$ kubectl config use-context user01-context
Switched to context "user01-context".
```

④ 명령어 테스트
```
$ kubectl get pod --user user01
Error from server (Forbidden): pods is forbidden: User "user01" cannot list resource "pods" in API group "" in the namespace "frontend"
```

user는 존재하되, `get pod`를 할 권한이 없기 때문에 Forbidden 에러가 발생하는 것이 정상이다.

⑤ usee01 -> admin 계정으로 다시 switch
```
 kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```