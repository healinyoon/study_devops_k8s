# Kubernetes Cluster

Kuberentes Cluster는 쿠버네티스의 여러 리소스를 관리하기 위한 집합체를 말한다.  
쿠버네티스 클러스터는 다음과 같이 Master와 Worker node로 구성된다.


쿠버네티스 마스터 서버에 배포되는 관리 컴포넌트는 다음과 같은 것이 있다.

| Component 명 | 역할 |
| --- | --- |
| kube-apiserver | * 쿠버네티스 API를 노출하는 kubectl로 부터 리소스를 조작하라는 지시를 받는다. <br/>* 쿠버네티스 관리 컴포넌트는 서로 직접 통신하지 않고 kube-apiserver를 거쳐서 통신한다. <br/>*Etcd와 통신하는 유일한 컴포넌트이다.|
| etcd | * 고가용성 분산 key-value 저장소 |
| kube-scheduler | * Node를 모니터링하고 Container를 배치할 적절한 Node를 선택한다. <br/>* 다수의 Pod를 배치하는 경우 Round-Robin을 사용하여 분산한다. |
| kube-conrtoller-manager | * 리소스를 제어하는 컨트롤러를 실행한다. <br/>* apiserver를 통해 받아진 요청을 '직접' 처리하는 역할 |


* 관리 Component 확인하기
```
$ kubectl get pod -n kube-system
```

* 경로
`/etc/kubernetes/manifests/` 경로에 kube-apiserver, kube-controller-manager, kube-etcd 등의 YAML이 저장되어 있다.