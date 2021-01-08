# Application Variables 관리의 필요성

단순한 변수가 변경될 때마다 application 버전을 변경하는 것은 낭비!
=> 변수를 사용하여 application 버전을 변경하지 않으면서(= image를 파괴하지 않으면서) 내용을 바꿀 수 있게 한다.

# 환경 변수 관리하기

### 환경 변수를 전달하는 방법

kubernetes의 환경 변수를 전달하는 방법은 크게 2가지로 나뉜다.
1. YAML `env` object에 직접 key-value로 전달하는 방법
2. Pod 외부의 리소스로 환경 변수를 지정하고 전달하는 방법

각 방법은 아래의 장/단점을 가진다.

1. YAML에서 설정하는 경우
    * 설정하기 쉽다.
    * 하드 코딩된 환경 변수는 여러 환경에 데이터를 정의, 유지, 관리가 어렵다.
2. Pod 외부의 리소스로 환경 변수를 지정하고 전달하는 경우
    * 별도의 리소스를 두어야 하지만,
    * `configMap` 또는 `Secret`을 사용하면 정의, 유지, 관리가 용이하다.

### 환경 변수 설정 예시

##### Type1. YAML에 key와 value로 지정
```
env:
- name: DEMO_GREETING
  value: "Hello k8s env"
```

##### Type2. ConfigMap
```
- name: DEMO_GREETING
  valueFrom:
    configMapKeyRef: configmap-name     <-- 외부에서 파일 참조
```

##### Type3. Secrets
```
- name: DEMO_GREETING
  valueFrom:
    secretKeyRef: secret-name           <-- 외부에서 파일 참조
```

이제부터 위의 세가지 Type을 사용하여 환경 변수를 지정하는 방법을 자세히 알아보자.

# 환경 변수 설정 방법 type 1) YAML의 key-value

### 개요

[※ Define Environment Variables 쿠버네티스 공식 문서](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

* 환경 변수를 지정하는 가장 간편한 방법이다.
* YAML 파일을 계속 업데이트 해줘야하는 불편함이 있다 => Pod를 계속 재시작 해야하는 것

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

