# Type 3) Secret으로 환경 변수 설정

### 개요

configMap이 평문으로 데이터를 저장하는 반면에, secret은 **인코딩된 상태(base64)**로 데이터를 저장해 둔다.
  * 비밀번호, OAuth token, ssh key와 같은 민감 정보에 적합하다.
  * 하지만 디코딩이 가능하기 때문에.. 공격자에게 혼란을 주는 정도이고 완벽한 보안이 된다고 보기는 어렵다.

Secret 데이터는 Etcd에 저장되며, 허가된 Pod에서만 조회 가능하다.
  * TLS 기반의 통신을 하고 있기 떄문에 가로채도 확인 불가능하다.
  * 당연히 권한이 있는 Pod나 사용자는 조회 가능하다.

Volume 마운트도 물론 가능하다.

# 사용 방법

### Command로 Secret을 생성하는 방법

#### 1. 환경 변수 파일 생성
```
$ echo -n 'admin' > ./username
$ echo -n '111222333' > ./password
```

#### 2. Secret 생성
```
$ kubectl create secret generic db-user-pass --from-file=./username --from-file=./password
secret/db-user-pass created
```
위의 명령어를 통해 `db-user-pass` Secret이 생성된다.

#### 3. 생성된 Secret 조회
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
  namespace: ``default``
  resourceVersion: "3509086"
  selfLink: /api/v1/namespaces/``default``/secrets/db-user-pass
  uid: 34db07f0-2631-4757-bd53-da5f0b82bef0
type: Opaque
```

#### (참고) 디코딩 해보고 싶다면
```
(형식)
$ echo {인코딩된 데이터} | base64 --decode

(예시)
$ echo MTExMjIyMzMz | base64 --decode
```

#### 4. 위에서 생성한 Secret을 사용하는 Pod YAML 작성

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

#### 5. Pod YAML 실행
```
$ kubectl create -f envar-secret.yaml
pod/secret-envar-demo created
```

#### 6. Pod bash에 접속해서 env 확인
```
$ kubectl exec -it secret-envar-demo -- bash
root@secret-envar-demo:/# printenv
PASS=111222333
USER=admin
```

### YAML 파일로 Secret을 생성하는 방법

수동으로 YAML 데이터를 만들어야 하는 경우에는 base64 인코딩을 수동으로 해서 넣어줘야 한다.

#### 1. 인코딩

> 예시
```
$ echo -n 'admin' | base64
YWRtaW4=

$ echo -n '111222333' | base64
MTExMjIyMzMz
```

#### 2. Secret YAML 파일 작성
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

# 연습 문제

### 문제 1

* kube-system에 존재하는 secret의 개수는 몇 개인가?
* ``default``-token의 개수는 몇 개인가?
* ``default``-token의 타입은 무엇인가?
* secret 데이터에는 어떤 secret data가 포함되어 있는가?
* 다음과 같이 Mysql 서버를 지원하는 secret-mysql.yaml을 생성하자.
  * secret name: db-secret  
  * secret data 1: DB_Password=Passw0rd!0

### 풀이 1

* kube-system에 존재하는 secret의 개수는 몇 개인가?
```
$ kubectl get secret -n kube-system | wc -l
36
```
=> 35개(맨 위의 줄은 col 명)

* ``default``-token의 개수는 몇 개인가?
```
$ kubectl get secret -n kube-system | grep ``default``
``default``-token-z9nhz                              kubernetes.io/service-account-token   3      29d
```
=> 1개

* ``default``-token의 타입은 무엇인가?
```
$ kubectl get secret ``default``-token-z9nhz -n kube-system -o yaml
apiVersion: v1
(중략)
type: kubernetes.io/service-account-token
```
=> service-account-token 타입

* secret 데이터에는 어떤 secret data가 포함되어 있는가?
```
$ kubectl get secret ``default``-token-z9nhz -n kube-system -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01Ea3dPREEzTXpRMU0xb1hEVE13TURrd05qQTNNelExTTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSytUCmxYTy9JWlRpbjdFMzNyWFVrT1RKSnNUUnBabTEydE53b2hFcFlyTnpjVU40NWw0SnhwdGxnZDYvbWJyekJNemsKNEhja1o1aWRGdUhPa0VSM3BEak5NRjZxUitZYkFhL2N4a3pUYlBCRTEyRmFPRWVqdE12Tm5pZmFQQmI5ZkJIWgpQdE5LUUtEOWk1clBGOXJPNDU0aHJxYjRabFZWSy80TWI1M1ZsZThMTGFmNDVYZWRsaWVUZk5Bbm15eHRNU1dHCjEvdVFxWG5XZmpmOVkxaExRVnRZd1QveEpyM2dnWWFBeUU5K0JaUzV6ak5tc1ZZa0g0ejFBcEZkYldlbTdUUDYKcTc2ZWVtbDluSTVzVWRKT2VleXdYS0p2TW14Qlg3VHNFUkhEMHdtTFB6VHJKTXIveXQ2cEl3SGRQUTBSTGNPTQplS2V4WVJCMHpudGg4VUE1Z2hjQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFNWl2MzdPZkIvc1IzcHdkZ1BCME5TT1FDalpNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDRklHeFR6WisvMHJlTHZXcFBvaGFKb2gvY0VXdVM4dXV0M2VnN2Zkd3FxMWVMTDJpRgpwWHdoWkN4ZWNEcmxUREJETEFlTXYvSnkvcFFJWi9rS1N4bXJmR1J3VGtnTnAyaUdlUS8vNXUzL1dEUWdFZUZ0CmY4ZG1hbjZCSEJpdGFFQlgrOS80NmdwclJEa3YzcTBLcTQxdTgwRU9DRDdtM0VCLzV3a1ZhTlFvMjEveFFVVjgKM2Rkd2xiZHhoVWF4Q0lFcnRvci9RNGthOFpRU2h4L3dxVE93c284QzhYZVlzMmVyOTJ2Q1VIMzBxdGdOaVZPeApMRzFBQmFkVk1Zc2ZuQjdvSGtyd0JNVWZVSDZFb1YrQUhZRVNJVnRHZ0xNQlF6U1VLTnVVNG0yVmp0cjR2dk9PCmpyaVFzVUV0TzAzd2hHcGxvWjhRNVpkNUovUmdSMktzTTdaWQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: a3ViZS1zeXN0ZW0=
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklrcGpUbGxXU2xFNGRrMXRObFprY25wR1JYTlRTM2hEUTJjMWNIaENVelJOWmt4ZmFVTmlNVVp5YVhjaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpyZFdKbExYTjVjM1JsYlNJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZqY21WMExtNWhiV1VpT2lKa1pXWmhkV3gwTFhSdmEyVnVMWG81Ym1oNklpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WlhKMmFXTmxMV0ZqWTI5MWJuUXVibUZ0WlNJNkltUmxabUYxYkhRaUxDSnJkV0psY201bGRHVnpMbWx2TDNObGNuWnBZMlZoWTJOdmRXNTBMM05sY25acFkyVXRZV05qYjNWdWRDNTFhV1FpT2lKaU5qSXdZekE0WkMwek5tVmxMVFF3TURNdFltTm1ZaTFsWVRjd1pEZ3pOR000T1RRaUxDSnpkV0lpT2lKemVYTjBaVzA2YzJWeWRtbGpaV0ZqWTI5MWJuUTZhM1ZpWlMxemVYTjBaVzA2WkdWbVlYVnNkQ0o5LkhQOXVVZzFyakpRVGZ6R3h4UThGb0NkMmdfOHp4eElkcndBRW5Tb1ZDSHZaSVg3ZkZURUNlb0NFWF94NDVFX2VBS190Wk5GaHlvaTNPZnBvdlNiMEZERXV4WFIwRzNRZTBJNWtmNEVLbXVJbXpzbWM5MGtCTVY1TEhLYXZJOTFmZHdtVlJzNUdtN0ItdnpBdXl3dXRERzZ2dVM0NDBzanQ1a0dfem1FcTFfSExjZmxWLWxESkhJVFNxU1k3YnBTcnkwMmxXT2ZGSGdfcWZqMmxoa3ppaGNaOVdrS2dubmNnaGZjX3Z3RDZ1d2lYbkotb3I0Q3IyRFZhU0lyRnpyX0Z5d2plVDFvTEsweXltc1lRejkwRHFoaG5DMUNITjFsMHZ4cDBmWjhJUER6WWhiY3BSWTF6dXFFc25SRUxiTWZkSFlVQzRhSFljRlVYTUlSZnh5RWFDQQ==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: ``default``
    kubernetes.io/service-account.uid: b620c08d-36ee-4003-bcfb-ea70d834c894
  creationTimestamp: "2020-09-08T07:35:43Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:ca.crt: {}
        f:namespace: {}
        f:token: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubernetes.io/service-account.name: {}
          f:kubernetes.io/service-account.uid: {}
      f:type: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-09-08T07:35:43Z"
  name: ``default``-token-z9nhz
  namespace: kube-system
  resourceVersion: "344"
  selfLink: /api/v1/namespaces/kube-system/secrets/``default``-token-z9nhz
  uid: 03519231-9aa2-43dc-9252-cd0dc4e2a091
type: kubernetes.io/service-account-token
```

위의 내용을 인코딩이 아닌 디코딩 된 형태로 보고 싶을 때는
```
$ kubectl describe secret ``default``-token-z9nhz -n kube-system
Name:         ``default``-token-z9nhz
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: ``default``
              kubernetes.io/service-account.uid: b620c08d-36ee-4003-bcfb-ea70d834c894

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkpjTllWSlE4dk1tNlZkcnpGRXNTS3hDQ2c1cHhCUzRNZkxfaUNiMUZyaXcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXo5bmh6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiNjIwYzA4ZC0zNmVlLTQwMDMtYmNmYi1lYTcwZDgzNGM4OTQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.HP9uUg1rjJQTfzGxxQ8FoCd2g_8zxxIdrwAEnSoVCHvZIX7fFTECeoCEX_x45E_eAK_tZNFhyoi3OfpovSb0FDEuxXR0G3Qe0I5kf4EKmuImzsmc90kBMV5LHKavI91fdwmVRs5Gm7B-vzAuywutDG6vuS440sjt5kG_zmEq1_HLcflV-lDJHITSqSY7bpSry02lWOfFHg_qfj2lhkzihcZ9WkKgnncghfc_vwD6uwiXnJ-or4Cr2DVaSIrFzr_FywjeT1oLK0yymsYQz90DqhhnC1CHN1l0vxp0fZ8IPDzYhbcpRY1zuqEsnRELbMfdHYUC4aHYcFUXMIRfxyEaCA
```

또는 직접 디코딩
```
namespace: a3ViZS1zeXN0ZW0=

위의 값을 디코딩하면 다음과 같이 확인할 수 있다.
$ echo a3ViZS1zeXN0ZW0= | base64 --decode
kube-system
```

### 문제 2

* Mysql 이미지를 하나 생성하고 앞서 만든 secret을 환경 변수 이름과 연결하자
  * Image: mysql:5.6
  * Port: 3306
  * 환경변수 name: MYSQL_ROOT_PASSWORD
* 잘 적용되었는지 확인할 수 있는 명령어
  * kubectl exec -it mysql --mysql -u root -p
  * password: Passw0rd!0

### 풀이 2

* 다음과 같이 Mysql 서버를 지원하는 secret-mysql.yaml을 생성하자.
  * secret name: db-secret  
  * secret data 1: DB_Password=Passw0rd!0

아래의 방식은 secret 데이터 파일을 만들지 않고 `--from=literal` 이라는 옵션을 통해 데이터를 받는다.
먼저 yaml 파일의 형식을 출력해보고
```
$ kubectl create secret generic db-secret --from-literal='DB_Password=Passw0rd!0' --dry-run=client -o yaml
apiVersion: v1
data:
  DB_Password: UGFzc3cwcmQhMA==
kind: Secret
metadata:
  creationTimestamp: null
  name: db-secret
```

`secret-mysql.yaml` 을 생성한다.
```
$ kubectl create secret generic db-secret --from-literal='DB_Password=Passw0rd!0' --dry-run=client -o yaml > secret-mysql.yaml
```

다음으로 생성한 YAML을 실행한다.
```
$ kubectl create -f secret-mysql.yaml
secret/db-secret created
```

* Mysql 이미지를 하나 생성하고 앞서 만든 secret을 환경 변수 이름과 연결하자
  * Image: mysql:5.6
  * Port: 3306
  * 환경변수 name: MYSQL_ROOT_PASSWORD

다음과 같이 mysql Pod YAML을 작성하며 앞서 만든 secret 환경 변수 적용
> mysql-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  containers:
  - name: mysql
    image: mysql:5.6
    ports:
    - containerPort: 3306
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_Password
```

Pod YAML 실행
```
$ kubectl create -f mysql-pod.yaml
pod/mysql created
```

* 잘 적용되었는지 확인할 수 있는 명령어
  * kubectl exec -it mysql --mysql -u root -p
  * password: Passw0rd!0

Pod의 mysql에 접속하여 환경 변수 적용 확인
```
$ kubectl exec -it mysql -- mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.49 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```