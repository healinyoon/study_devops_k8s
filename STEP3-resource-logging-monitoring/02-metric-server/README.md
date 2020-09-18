
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

