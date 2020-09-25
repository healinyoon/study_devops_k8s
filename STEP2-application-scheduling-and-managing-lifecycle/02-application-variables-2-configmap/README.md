# Type2. ConfigMap에 환경 변수를 저장하는 방법

[※ 쿠버네티스 configMap 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)

### 개요
* configMap은 환경 변수를 저장하기도 하지만, 그 외에도 다양한 기능이 있다(예: 스토리지에 파일 저장 등).
* configMap은 `kubectl create configmap {configMap 명} --from-file={file 명}...(여러 개 입력 가능)` 명령어를 통해 1) 생성할 수 있고 2) 특정 파일로부터 환경변수 값을 얻는다.
  * file key: 파일명
  * value: 파일 내용
* 쿠버네티스 리소스 생성시 해당 configMap을 참조하도록 설정해서 사용한다.
  * 쿠버네티스 리소스의 `spec.containers.env`의 `configMapKeyRef.name`과 configMap의 `metadata.name`을 매칭시킨다.
  * 쿠버네티스 리소스의 `spec.containers.env`의 `configMapKeyRef.key`와 configMap의 `data.{파일 명}`을 매칭시킨다.
  * 쿠버네티스 리소스의 `env.name`이 환경 변수 명, file에 저장된 값이 환경 변수 값이 된다.

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
  data: "1234"                              <--  다수의 file을 지정할 수도 있다('--from-file={file 명}'을 더 추가하면 됨).
kind: ConfigMap
metadata:
  creationTimestamp: "2020-09-16T08:41:54Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:data: {}
    manager: kubectl-create
    operation: Update
    time: "2020-09-16T08:41:54Z"
  name: test-configmap                       <-- configmap의 metadata.name
  namespace: default
  resourceVersion: "1662797"
  selfLink: /api/v1/namespaces/default/configmaps/test-configmap
  uid: 1865162a-b9ec-48a5-a680-79ff899e0d19
```

* 위에서 생성한 configMap을 사용하는 Pod YAML 작성

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
    - name: DEMO_GREETING               <-- 환경 변수 명
      valueFrom:
        configMapKeyRef:                <-- 아래를 통해 참조한 configmap을 통해 환경 변수 값을 얻는다.
          name: test-configmap          <-- configmap의 metadata.name과 매칭
          key: data                     <-- configmap의 data.{파일 명}과 매칭
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

### ConfigMap의 모든 key-value를 환경 변수로 지정하는 방법

[※ 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)

* configMap YAML 작성

> configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config                  <-- configmap의 metadata.name
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

* configMap YAML 실행
```
$ kubectl create -f configmap-multikeys.yaml
configmap/special-config created
```

* 생성된 configMap 리소스 확인
```
$ kubectl get configmap
NAME             DATA   AGE
special-config   2      32s
```

* 위에서 생성한 configMap을 사용하는 Pod YAML 작성

Use `envFrom` to define all of the ConfigMap's data as container environment variable

> pod-configmap-envFrom.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    envFrom:
    - configMapRef:
        name: special-config              <-- configmap의 metadata.name과 매칭
  restartPolicy: Never
```

* Pod YAML 실행
```
$ kubectl apply  -f pod-configmap-envFrom.yaml
pod/dapi-test-pod created
```

* 생성된 Pod 리소스 확인
```
$ kubectl get pod dapi-test-pod
NAME            READY   STATUS    RESTARTS   AGE
dapi-test-pod   1/1     Running   0          94s
```

* Pod bash에 접속해서 env 확인
```
$ kubectl exec -it dapi-test-pod  -- bash
root@dapi-test-pod:/# printenv | grep SPECIAL
SPECIAL_LEVEL=very
SPECIAL_TYPE=charm
```