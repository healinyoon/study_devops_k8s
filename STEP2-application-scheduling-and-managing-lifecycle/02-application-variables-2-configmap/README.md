# Type 2) ConfigMap로 환경 변수 설정

[※ 쿠버네티스 configMap 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)

### 개요
**configMap**이라는 쿠버네티스 리소스를 사용하여 환경 변수를 지정한다. configMap은 환경 변수를 저장하는데 사용되기도 하지만, 그 외에도 다양한 기능이 있다(예: 스토리지에 파일 저장 등).

configMap은 다음 명령어를 통해 생성할 수 있고, 특정 파일로부터 환경 변수 값을 얻는다. 

```
kubectl create configmap {configMap 명} --from-file={file 명}...(여러 개 입력 가능)
```

쿠버네티스 리소스를 구성할 때 미리 생성한 configMap을 참조하도록 설정해서 사용한다.

```
env:
- name: DEMO_GREETING               <-- 환경 변수 명
  valueFrom:
    configMapKeyRef:                <-- 환경변수를 configmap 타입으로 전달받겠다.
      name: test-configmap          <-- configmap의 metadata.name과 매칭
      key: data                     <-- configmap의 data.{file 명}과 매칭
```
이렇게 설정하면 쿠버네티스 리소스의 `env.name`이 환경 변수 명, configmap의 `data.{file 명}`에 저장된 값이 환경 변수 값이 된다.

### 사용 방법

#### 1. 값이 저장된 파일 생성
```
$ echo -n 1234 > data
```

#### 2. 파일을 참조하는 configMap 생성
```
$ kubectl create configmap test-configmap --from-file=data
configmap/test-configmap created
```
다수의 file을 지정할 수도 있다('--from-file={file 명}'을 더 추가하면 됨).

#### 3. 생성된 configMap 조회
```
$ kubectl get configmap test-configmap -o yaml
apiVersion: v1
data:
  data: "1234"                              
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

#### 4. configMap을 참조하는 Pod YAML 작성

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
        configMapKeyRef:                <-- 환경변수를 configmap 타입으로 전달받겠다.
          name: test-configmap          <-- configmap의 metadata.name과 매칭
          key: data                     <-- configmap의 data.{file 명}과 매칭
```

#### 5.Pod YAML 실행
```
$ kubectl create -f configmap-envar-demo.yaml
pod/configmap-envar-demo created
```

#### 6. 생성된 Pod 리소스 확인
```
$ kubectl get Pod
NAME                       READY   STATUS    RESTARTS   AGE
configmap-envar-demo       1/1     Running   0          35s
```

#### 7. Pod bash에 접속해서 env 확인
```
$ kubectl exec -it configmap-envar-demo -- bash
root@configmap-envar-demo:/# printenv | grep DEMO_GREETING
DEMO_GREETING=1234
```

# ConfigMap의 모든 key-value를 환경 변수로 지정하는 방법

### 개요

위에서 처럼 configMap에 매칭된 file을 하나씩 가져와서 설정하지 않고, configMap에 저장된 모든 file을 환경 변수로 한번에 지정하는 방법이다.

[※ 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables)

![](/STEP2-application-scheduling-and-managing-lifecycle/images/02-application-variables-2-configmap-1.png)  

### 사용 방법

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

# ConfigMap을 활용한 디렉토리 마운트

### 개요

[※ 쿠버네티스 Add ConfigMap data to a Volume 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)

디렉토리 마운트를 통한 환경 변수 설정 방법에 대해 설명한다.

#### 중요한 장점!!!
이전에 설정한 방법들은 Container를 재시작해야만 환경 변수가 변경되는데,  
volume을 통해 설정하는 경우 약 1분마다 데이터 값이 refresh되어 적용되므로, Container 외부에서도 환경 변수를 설정할 수 있게 된다.

### 사용 방법

* configMap YAML 작성

> configmap-multikeys.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

* configMap YAML 실행
여기서는 k8s 공식 문서의 링크를 파라미터로 전달하여 사용했다. 위의 YAML 내용과 동일하다.
```
$ kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml
configmap/special-config created
```

* configMap YAML 확인
```
$ kubectl get configmaps special-config -o yaml
apiVersion: v1
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
kind: ConfigMap
metadata:
  creationTimestamp: "2020-09-25T06:07:23Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:SPECIAL_LEVEL: {}
        f:SPECIAL_TYPE: {}
    manager: kubectl-create
    operation: Update
    time: "2020-09-25T06:07:23Z"
  name: special-config
  namespace: default
  resourceVersion: "3499583"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 4d244acd-4265-406b-b739-d2623b33d83c
```

* 위에서 생성한 configMap을 사용하는 Pod YAML 작성
```
apiVersion: v1
kind: Pod
metadata:
  name: volumes-dapi-test-pod
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config            <-- 이 경로에 special-config의 파일을 마운트 하겠다는 의미
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
```

* Pod YAML 실행
```
$ kubectl create -f pod-volumes-configmap.yaml
pod/volumes-dapi-test-pod created
```

* 생성된 Pod 리소스 확인
```
$ kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
volumes-dapi-test-pod   1/1     Running   0          9s
```

* Pod bash에 접속
```
$ kubectl exec -it volumes-dapi-test-pod -- bash
root@volumes-dapi-test-pod:/#
```

여기서 `printenv`를 하면 안된다. 왜냐하면 환경 변수로 저장하지 않고, `/etc/config/` 경로에 저장해두었기 때문이다.

해당 경로에 접근하면 다음과 같이 파일이 마운트 된 것을 확인 할 수 있다.
```
root@volumes-dapi-test-pod:/# cd /etc/config/
root@volumes-dapi-test-pod:/etc/config# ls
SPECIAL_LEVEL  SPECIAL_TYPE
```

