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

> 참고
그러나 working set 특정은 호스트 OS에 따라 다르며, 일반적으로 heuristics를 사용해서 평가한다. Kubernetes는 swap을 지원하지 않기 때문에 모든 익명(non-file-backed) 메모리를 포함한다. 호스트 OS가 항상 이런 페이지를 회수할 수 없기 때문에 메트릭에는 일반적으로 일부 캐시된(file-backed) 메모리도 포함된다. 

# Metric Server 설치 방법

[※ 참고 문서](https://blog.naver.com/isc0304/221860790762)

# metric-server 설치

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

