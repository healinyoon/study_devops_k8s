# Helm Chart를 활용한 Kubernetes 모니터링 시스템 EFK 만들기

### EFK 개요

EFK는 Elasticsearch + Logstash + Kibana를 의미한다.

* E: Elasticsearch 데이터 베이스 & 검색 엔진
* F: Fluentd 데이터 수집기 & 파이프라인 ← daemonSet으로 배치
* K: Kibana 대시보드

Kubernetes에서 EFK를 사용하면 각 Pod의 log를 수집하고 분석 및 시각화 할 수 있다.  
⟺ (참고) Prometheus + Grafana는 주로 메트릭을 수집해서 성능을 모니터링 한다.


### Helm Chart로 EFK 설치

#### 1. Helm Repo 추가 및 확인

* Chart repo 추가
```
$ helm repo add akomljen-charts \
>     https://raw.githubusercontent.com/komljen/helm-charts/master/charts/
"akomljen-charts" has been added to your repositories
```

* Chart repo 조회
```
$ helm repo list
NAME           	URL
stable         	https://charts.helm.sh/stable
akomljen-charts	https://raw.githubusercontent.com/komljen/helm-charts/master/charts/
```

#### 2. EFK Namespace 생성

```
$ kubectl create ns efk-logging
namespace/efk-logging created
```

#### 3. es-operator 설치

* es-operator 설치
```
$ helm install es-operator --namespace efk-logging akomljen-charts/elasticsearch-operator
NAME: es-operator
LAST DEPLOYED: Tue Jan 12 08:26:13 2021
NAMESPACE: efk-logging
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

* es-operator 조회
```
$ helm ls --namespace=efk-logging
NAME       	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART                       	APP VERSION
es-operator	efk-logging	1       	2021-01-12 08:26:13.219956144 +0000 UTC	deployed	elasticsearch-operator-0.1.7	0.3.0


$ kubectl get all -n efk-logging
NAME                                                      READY   STATUS    RESTARTS   AGE
pod/es-operator-elasticsearch-operator-575d9966bc-rjqk5   1/1     Running   0          63s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/es-operator-elasticsearch-operator   1/1     1            1           63s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/es-operator-elasticsearch-operator-575d9966bc   1         1         1       63s
```

* 사용자 정의 리소스가 잘 추가되었는지 확인
```
$ kubectl get CustomResourceDefinition
NAME                                         CREATED AT
elasticsearchclusters.enterprises.upmc.com   2021-01-12T08:18:38Z
```

#### 4. EFK 설치