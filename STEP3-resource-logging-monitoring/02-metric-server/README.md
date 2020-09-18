# Resource Metric Pipeline

[※ 쿠버네티스 Resource metric pipeline 공식 문서](https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-metrics-pipeline/)

### 개요

Container CPU 및 Memory 사용량과 같은 리소스 사용량 메트릭은 쿠버네티스의 메트릭 API를 통해 사용할 수 있다. 이 메트릭은 `kubectl top` 명령어 사용과 같이 사용자가 직접 액세스하거나, `Horizontal Pod Autoscaler` 같은 클러스터의 컨트롤러에서 결정을 내릴 때 사용될 수 있다.

### Metric API

메트릭 API를 통해 주어진 Node나 Pod에서 현재 사용 중인 리소스의 양을 알 수 있다. 이 API는 메트릭 값을 저장하지 않으므로, Node에서 10분 전에 사용된 리소스의 양을 가져오는 것과 같은 작업은 불가능하다.

* 다른 Kubernetes API의 엔드포인트와 같이 `/apis/metrics.k8s.io` 하위 경로에서 발견 될 수 있다.
* 이 API를 사용하려면 metric server를 클러스터에 배포해야한다.

### 리소스 사용량 측정

#### CPU 측정

일정기간 동안 CPU 코어의 평균 사용량을 측정한다. 이 값은 커널에서 제공하는 누적 CPU 카운터보다 높은 비율을 적용해서 얻는다.

#### Memory 측정

memory는 메트릭이 수집된 순간의 working set 사용량을 측정한다. 여기서 working set이란 'memory pressure'에서 풀려날 수 없는 'in-use' 상태를 의미한다. 

> (참고) Working set 특정은 호스트 OS에 따라 다르며, 일반적으로 heuristics를 사용해서 평가한다. Kubernetes는 swap을 지원하지 않기 때문에 모든 익명(non-file-backed) 메모리를 포함한다. 호스트 OS가 항상 이런 페이지를 회수할 수 없기 때문에 메트릭에는 일반적으로 일부 캐시된(file-backed) 메모리도 포함된다. 

# Metric Server 설치 방법

[※ Metric Server 설치 참고 문서](https://blog.naver.com/isc0304/221860790762)

### metric-server 설치

* metric-server repository 다운로드

```
$ git clone https://github.com/kubernetes-sigs/metrics-server
Cloning into 'metrics-server'...
remote: Enumerating objects: 49, done.
remote: Counting objects: 100% (49/49), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 12248 (delta 13), reused 23 (delta 3), pack-reused 12199
Receiving objects: 100% (12248/12248), 12.47 MiB | 6.87 MiB/s, done.
Resolving deltas: 100% (6389/6389), done.
```

* metric-server YAML 실행

README.md를 읽어보면 다음과 같이 설치 방법을 안내하고 있다.
```
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
Warning: apiregistration.k8s.io/v1beta1 APIService is deprecated in v1.19+, unavailable in v1.22+; use apiregistration.k8s.io/v1 APIService
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

* metric-server 테스트
```
$ kubectl top nodes
error: metrics not available yet
```

여기까지 실행하면 metric-server는 동작하지만 kubelet에 접근해서 Pod와 Node의 정보를 얻어오지는 못한다.  

* metric-server 내용 수정
이는 TLS 통신이 제대로 이루어지지 않기 때문이므로, 다음과 같이 metric-server의 내용을 수정하여 서버 통신이 되도록한다.

```
$ kubectl edit deployments.apps -n kube-system metrics-server

spec.containers.args Array에 아래 두 개의 옵션 추가

- --kubelet-insecure-tls: 인증서가 공인 기관에 승인을 받지 않은 인증서이지만 무시한다.
- --kubelet-preferred-address-types=InternalIP: kebelet 연결에 사용할 주소 타입 지정
```

변경된 내용
```
(중략)
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
(중략)
```

### metric-servert 사용
정보 수집을 위한 시간이 필요하므로 1분 정도 지난 후 Pod와 Node의 리소스 조회를 요청하면 정상적으로 모니터링 할 수 있다. 

* Node 조회
```
$ kubectl top nodes
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master    337m         16%    4484Mi          57%
worker1   77m          3%     1458Mi          18%
worker2   57m          2%     2399Mi          30%
```


* default Namespace의 Pod 조회
```
$ kubectl top pod
NAME                       CPU(cores)   MEMORY(bytes)
configmap-envar-demo       0m           11Mi
envar-demo                 0m           11Mi
http-go-5c6f458dc9-m97w8   0m           2Mi
```

* kube-system Namespace의 Pod 조회
```
$ kubectl top pod -n kube-system
NAME                              CPU(cores)   MEMORY(bytes)
coredns-f9fd979d6-79vc6           3m           14Mi
coredns-f9fd979d6-gfbpg           4m           14Mi
etcd-master                       16m          64Mi
kube-apiserver-master             40m          293Mi
kube-controller-manager-master    12m          51Mi
kube-proxy-49zfz                  1m           21Mi
kube-proxy-587jv                  1m           18Mi
kube-proxy-8p6v6                  1m           17Mi
kube-scheduler-master             4m           19Mi
metrics-server-75f98fdbd5-mfb2k   1m           14Mi
weave-net-p964j                   1m           67Mi
weave-net-q6vj7                   1m           69Mi
weave-net-v5qqm                   2m           65Mi
```