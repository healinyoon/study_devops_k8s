# 환경 변수 설정 방법 type 3) Secret

### 개요

* configMap이 평문으로 데이터를 저장하는 반면에, **인코딩된 상태(base64)**로 데이터를 저장해 둔다.
  * 비밀번호, OAuth token, ssh key와 같은 민감 정보에 적합하다.
  * 하지만 디코딩이 가능하기 때문에.. 공격자에게 혼란을 주는 정도이고 완벽한 보안이 된다고 보기는 어렵다.
* Secret 데이터는 Etcd에 저장되며, 허가된 Pod에서만 조회 가능하다.
  * TLS 기반의 통신을 하고 있기 떄문에 가로채도 확인 불가능하다.
  * 당연히 권한이 있는 Pod나 사용자는 조회 가능하다.

# 사용 방법

### Command로 Secret을 생성하는 방법

* 환경 변수 파일 생성
```
$ echo -n 'admin' > ./username
$ echo -n '111222333' > ./password
```

* Secret 생성
```
$ kubectl create secret generic db-user-pass --from-file=./username --from-file=./password
secret/db-user-pass created
```
위의 명령어를 통해 `db-user-pass` Secret이 생성된다.

* Secret 조회
```
$ kubectl get secret db-user-pass -o yaml
apiVersion: v1
data:
  password: MTExMjIyMzMz    <-- 자동으로 인코딩 되어 있음
  username: YWRtaW4=        <-- 자동으로 인코딩 되어 있음
kind: Secret
metadata:
  creationTimestamp: "2020-09-25T07:13:36Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:password: {}
        f:username: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2020-09-25T07:13:36Z"
  name: db-user-pass
  namespace: default
  resourceVersion: "3509086"
  selfLink: /api/v1/namespaces/default/secrets/db-user-pass
  uid: 34db07f0-2631-4757-bd53-da5f0b82bef0
type: Opaque
```

* 디코딩 해보고 싶다면
```
(형식)
$ echo {인코딩된 데이터} | base64 --decode

(예시)
$ echo MTExMjIyMzMz | base64 --decode
```

* 위에서 생성한 Secret을 사용하는 Pod YAML 작성

> envar-secret.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: USER
      valueFrom:
        secretKeyRef:            <-- db-user-pass라는 Secret에 가서 username을 읽어온다
          name: db-user-pass
          key: username
    - name: PASS
      valueFrom:
        secretKeyRef:            <-- db-user-pass라는 Secret에 가서 password를 읽어온다
          name: db-user-pass
          key: password
```

* Pod YAML 실행
```
$ kubectl create -f envar-secret.yaml
pod/secret-envar-demo created
```

* Pod bash에 접속해서 env 확인
```
$ kubectl exec -it secret-envar-demo -- bash
root@secret-envar-demo:/# printenv
PASS=111222333
USER=admin
```

### YAML 파일로 Secret을 생성하는 방법

수동으로 YAML 데이터를 만들어야 하는 경우에는 base64 인코딩을 수동으로 해서 넣어줘야 한다.

* 인코딩

> 예시
```
$ echo -n 'admin' | base64
YWRtaW4=

$ echo -n '111222333' | base64
MTExMjIyMzMz
```

* Secret YAML 파일 작성
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Qpaque
data:
  username: YWRtaW4=
  password: MTExMjIyMzMz
```
