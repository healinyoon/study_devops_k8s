# Application Variables 관리의 필요성

단순한 변수가 변경될 때마다 application 버전을 변경하는 것은 낭비!
=> 변수를 사용하여 application 버전을 변경하지 않으면서(= image를 파괴하지 않으면서) 내용을 바꿀 수 있게 한다.

# 환경 변수 관리하기

### 환경 변수를 전달하는 방법

kubernetes의 환경 변수를 전달하는 방법은 크게 2가지로 나뉜다.
하나는 YAML `env` object에 직접 key-value로 전달하는 방법이고, 다른 하나는 다른 리소스로 전달하는 방법이다.

* YAML에서 설정하는 경우
  * 하드 코딩된 환경 변수는 여러 환경에 데이터를 정의, 유지, 관리가 어렵다.
* 다른 리소스를 통해 전달하는 경우
  * Pod 외부의 리소스로 환경 변수 지정하는 방법이다.
  * `configMap` 또는 `Secret`을 사용하면 관리가 용이하다.

### 환경 변수 설정 예시

* Type1. YAML에 key와 value로 지정
```
env:
- name: DEMO_GREETING
  value: "Hello k8s env"
```

* Type2. ConfigMap
```
- name: DEMO_GREETING
  valueFrom:
    configMapKeyRef: configmap-name     <-- 외부에서 파일 참조
```

* Type3. Secrets
```
- name: DEMO_GREETING
  valueFrom:
    secretKeyRef: secret-name           <-- 외부에서 파일 참조
```

# Type1. YAML에 key와 value로 지정하는 방법

### 개요

[※ Define Environment Variables 쿠버네티스 공식 문서](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

* 가장 간편한 방법이다.
* YAML 파일을 계속 업데이트 해줘야하는 불편함이 있다.

### 사용 방법

* YAML 작성 요령

> envar-demo.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```

* YAML 실행
```
$ kubectl create -f envar-demo.yaml
pod/envar-demo created
```

* 생성된 Pod 리소스 확인
```
$ kubectl get podNAME                       READY   STATUS    RESTARTS   AGE
envar-demo                 1/1     Running   0          79s
```

* Pod bash에 접속해서 env 확인
```
$ kubectl exec -it envar-demo -- bash

root@envar-demo:/# printenv
(중략)
DEMO_FAREWELL=Such a sweet sorrow           <-- YAML에 설정한 환경 변수 확인
DEMO_GREETING=Hello from the environment    <-- YAML에 설정한 환경 변수 확인
```

# Type2. ConfigMap에 환경 변수를 저장하는 방법

### 개요
* configMap은 환경 변수를 저장하기도 하지만, 그 외에도 다양한 기능이 있다(예: 스토리지에 파일 저장 등).
* configMap은 `kubectl create configmap {configMap 명} --from-file={file 명}...(여러 개 입력 가능)` 명령어를 통해 1)생성할 수 있고 2)특정 파일로부터 key와 value를 얻는다.
  * file key: 파일명
  * value: 파일 내용
* 쿠버네티스 리소스 생성시 해당 configMap을 참조하도록 설정해서 환경 변수를 사용한다.
  * 쿠버네티스 리소스의 `spec.containers.env`의 `configMapKeyRef.name`과 configMap의 `metadata.name`을 매칭시키면 참조 가능하다.

### 사용 방법

* 데이터 파일 생성
```
$ echo -n 1234 > data
```

* configMap 생성
```
$ kubectl create configmap test-configmap --from-file=data
configmap/test-configmap created
```

* 생성된 configMap 조회
```
$ kubectl get configmap test-configmap -o yaml
apiVersion: v1
data:
  data: "1234"      <--  다수의 file key를 저장할 수도 있다('--from-file={file 명}'을 더 추가하면 됨).
kind: ConfigMap
(중략)
```

* configMap을 사용하는 Pod YAML 작성

> configmap-envar-demo.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      valueFrom:
        configMapKeyRef:
          name: test-configmap          <-- configmap의 metadata.name 매칭
          key: data
```

* YAML 실행
```
$ kubectl create -f configmap-envar-demo.yaml
pod/configmap-envar-demo created
```

* 생성된 Pod 리소스 확인
```
$ kubectl get Pod
NAME                       READY   STATUS    RESTARTS   AGE
configmap-envar-demo       1/1     Running   0          35s
```

* Pod bash에 접속해서 env 확인
```
$ kubectl exec -it configmap-envar-demo -- bash
root@configmap-envar-demo:/# printenv | grep DEMO_GREETING
DEMO_GREETING=1234
```