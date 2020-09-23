# Label 이란?

### 개요

* 모든 리소스를 구성하는 매우 간단하면서도 강력한 k8s 기능이다.
* 리소스에 첨부하는 임의의 key-value 쌍 값
* Label selector를 사용하면 각종 리소스를 필터링하여 선택할 수 있다.
* 리소스는 한 개 이상의 Label을 가질 수 있다.
* 리소스를 만드는 시점에 Label을 작성한다.
* 기존에 실행 중인 리소스에도 Label 값을 수정 및 추가 가능하다.

### Label을 이용한 Pod 구성

용도에 따라 Label을 구분하여 사용함  
![](/STEP1-core-concepts-of-k8s/images/04-Label-1.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터

# Label 사용 방법

### Pod 생성 시 Label 지정 방법

`labels` 오브젝트에 `key:value` 형식으로 지정

> http-go-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: http-go
  # label 추가
  labels:
    creation_method: manual
    env: prod
spec:
  containers:
  - name: http-go
    image: healinyoon/http-go
    ports:
    - containerPort: 8080
      protocol: TCP

```

### Label을 추가 및 수정하는 방법

* 새로운 Label을 추가할 때는 `label` 명령어 사용
```
$ kubectl label pod {Pod 명} {key}={value}
```

* 기존의 실행 중인 리소스의 Label을 수정할 때는 `--overwrite` 옵션을 주어서 실행
```
$ kubectl label pod {Pod 명} {key}={value} --overwrite
```

* Label 삭제
```
$ kubectl label pod {Pod 명} {Label key}-
```

# Label 배치 전략

확장 가능한 k8s Label 예제
![](/STEP1-core-concepts-of-k8s/images/04-Label-2.png)  
이미지 출처: 인프런-데브옵스를 위한 쿠버네티스 마스터