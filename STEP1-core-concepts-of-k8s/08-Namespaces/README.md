# Namespace 란?

* Resources를 분리된 영역으로 나눌 수 있는 방법
* 다수의 Namespaces를 사용하여 복잡한 쿠버네티스 시스템을 더 작은 그룹으로 분할
  * Namespaces 내에서 resource 이름을 고유하게 사용 가능
  * Resources를 생산, 개발, QA 등 환경으로 분리하여 사용 가능
* 다른 사용자와 분리된 환경으로 타인의 접근 제한 가능


현재 cluster의 기본 namespace 확인 방법:
```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   3d18h
kube-node-lease   Active   3d18h
kube-public       Active   3d18h
kube-system       Active   3d18h
```

**Tip!**  
지금까지 사용한 `kubectl delete all -all` 명령어도 `default` namespace에서만 수행되었음

# Namespace 사용 방법

### 기본 형식

`kubectl get --namespace {namespace 명}` 또는 `kubectl get --n {namespace 명}`으로 질의  

옵션 없이 사용하면 default namespace에 질의

### 전체 Namespace 조회하는 방법(관리자를 위한 명령어)

```
$ kubectl get pod --all-namespaces
```

# Namespace 생성 방법

### 방법 1) yaml 파일 작성 후 `kubectl create -f {yaml 파일 명}` 명령어 수행

- 명령어
```
$ kubectl create -f test-namespace.yaml
```

- yaml 파일 형식
```
apiVersion: apps/v1
kind: Namespace
metadata:
  # Namespace 명
  name: test-namespace
```

### 방법 2) `kubectl` 명령어로 바로 생성

```
$ kubectl create namespace test-namespace
```

