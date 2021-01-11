# Pod의 수동 scheduling: 원하는 Pod를 원하는 Node에

### 개요

특정 Node에 종속되어 실행되도록 Pod를 제한한다. 일반적으로 scheduler는 자동으로 합리적인 배치를 수행하므로 이러한 제한이 불필요하지만, 다음과 같은 케이스에서 사용될 수 있다.

* SSD가 있는 Node에서 Pod가 실행되어야 하는 경우
* 블록체인이나 딥러닝 시스템을 위해 GPU가 필요한 경우
* 서비스 성능의 극대화를 위해 하나의 Node에 서비스에 필요한 Pod를 모두 배치하는 경우

### 방법 1. nodeName을 활용한 scheduling

* Pod를 강제로 원하는 Node에 manually scheduling
* `nodeName` object에 지정

#### step 1. Node Name 조회
```
# kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   124d   v1.19.0
worker1   Ready    <none>   124d   v1.19.0
worker2   Ready    <none>   124d   v1.19.0
```

#### step 2. Node Selector 지정

> nodeName.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  containers:
  - name: http-go
    image: gasbugs/http-go
  nodeName: {node name}
```


### 방법 2. nodeSelector을 활용한 scheduling

* 특정 하드웨어를 가진 Node에서 Pod를 실행하고자 하는 경우 적용
* GPU, SSD 등의 이슈를 가진 사항을 적용
* Node에 label 지정 후 `nodeSelector` object에 지정

#### step 1. Node에 label 지정
```
# kubectl label node {node name} {label key=value}
# kubectl get node --show-labels
```

#### step 2. Node Selector 지정

> nodeSelector.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  containers:
  - name: http-go
    image: gasbugs/http-go
  nodeSelector:
    {key}: "{value}"
```