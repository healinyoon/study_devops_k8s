# Istio

[Istio 공식 홈페이지](https://istio.io/latest/docs/concepts/what-is-istio/)

### Istio란?

Kubernetes는 container를 편하게 관리할 수 있는 다양한 이점을 제공한다. 그러나 Kubernetes 만으로는 각 container의 트래픽을 관찰하고 정상 동작하는지 모니터링하기 어려운데, 이는 DevOps 팀에 부담이 될 수 있다. Service mesh의 크기와 복잡성이 증가할 수록 더욱 이해하고 관리하기가 어려워진다(예: 검색, 로드밸런싱, 장애 복구, 메트릭, 모니터링 등). Istio는 이런 Kubernetes 환경의 전체 service mesh 동작에 대한 통찰력과 운영 제어를 제공해준다.

> Service mesh란? Application과 이들 간의 상호 작용을 구성하는 마이크로 서비스 네트워크 구조

### Istio를 사용해야 하는 이유
* Kubernetes의 복잡성을 감소시킬 수 있다(Istio는 Kubernetes외에도 다양한 플랫폼에서 사용되는 service mesh 관리를 위한 프로젝트이다). 
* 서비스의 연결, 보안, 제어 및 관찰 기능을 구현할 수 있다. 
* 모든 container를 로깅하거나, 원격 측정, 정책 시스템을 통합할 수 있는 API 플랫폼이다.
* 설정이 간편하다.
    * 서비스의 code를 거의 변경하지 않고도 namespace에 설정 옵션을 주면 바로 사용 가능하다.
    * micro service간의 모든 네트워크 통신을 관찰하는 특수 사이드카 proxy를 배치해서, 각 pod에 istio-proxy 사이드카를 통해 모니터링 가능하다.

### Istio의 기능

| 기능 | 설명 |
| --- | --- |
| 연결 | 서비스 간의 트래픽 및 API 호출 흐름을 지능적으로 제어하고, 다양한 테스트를 수행하며 Red/Black 배포를 통해 점진적으로 업그레이드 |
| 보안 | 관리 인증, 권한 부여 및 서비스 간 통신 암호화를 통해 서비스를 자동으로 보호 |
| 제어 | 정책을 제어하고 실행, 소비자에게 공정하게 분배 |
| 관찰 | 모든 서비스의 풍부한 자동 추적, 모니터링 및 로깅으로 발생 상황 확인 |


# Istio 설치

[Istio 다운로드 경로](https://istio.io/latest/docs/setup/getting-started/#download)

### Istioctl 최신 버전 다운로드 및 설치

#### 1. 설치 파일 다운로드
```
$ curl -L https://istio.io/downloadIstio | sh -
```

#### 2. 설치 경로로 이동
```
$ cd istio-1.8.1
```

#### 3. 환경 변수 설정
```
$ export PATH=$PWD/bin:$PATH
```

### Istio 설치 프로파일

Istio를 설치할 때 원하는 프로파일을 지정하여 구성할 수 있다.

| 프로파일 | default | demo | minimal | remote | empty | preview |
| --- | --- | --- | --- | --- | --- | --- |
| Core Componentes | | | | | | |
| istio-egressgateway | | O | | | | |
| istio-ingressgateway | O | O | | | | O |
| istiod | O | O | O | | | O |


### istioctl을 활용한 istio 설치

istio 설치를 이어서 진행한다. `$ istioctl install --set profile={프로파일 타입} -y` 명령어를 사용하여 간단하게 설치할 수 있다.

#### 1. 프로파일 조회

```
$ istioctl profile list
Istio configuration profiles:
    default
    demo
    empty
    minimal
    openshift
    preview
    remote
```

#### 2. demo 프로파일 설치

```
$ istioctl install --set profile=demo -y
Detected that your cluster does not support third party JWT authentication. Falling back to less secure first party JWT. See https://istio.io/v1.8/docs/ops/best-practices/security/#configure-third-party-service-account-tokens for details.
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete
```

여기까지 진행하면 istio를 사용할 수 있는 환경 설정이 완료된다.

#### 3. Namespace lable 설정

Istio 설치가 완료되면 Namespace 마다 Istio를 적용할지 설정할 수 있다. Namespace에 `istio-injection=enabled` label을 추가하면 Istio가 바로 적용된다.

```
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

### Sample 프로젝트 Application 생성

[Sample 프로젝트(Book Info)](https://istio.io/latest/docs/examples/bookinfo/)

Sample 프로젝트 `bookinfo`를 사용하여 Istio를 사용해보자.

![](/STEP4-istio/images/bookinfo.svg)

그림 출처: https://istio.io/latest/docs/examples/bookinfo/

```
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```

```
$  kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79c697d759-85bfd       2/2     Running   0          2m55s
productpage-v1-65576bb7bf-l7csc   2/2     Running   0          2m53s
ratings-v1-7d99676f7f-7x8nl       2/2     Running   0          2m54s
reviews-v1-987d495c-r52wh         2/2     Running   0          2m53s
reviews-v2-6c5bf657cf-n5vwl       2/2     Running   0          2m54s
reviews-v3-5f7b9f4f77-2fpzb       2/2     Running   0          2m54s
```

### Gateway 설치와 관찰

위에서 Application을 생성했지만, Gateway를 추가로 생성해야 외부에서 접근 가능해진다. Gateway는 Istio가 배포한 커스터마이징된 object로, Kubernetes에서 제공되는 Ingress 처럼 동작한다.

#### 1. Gateway 설치

```
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

#### 2. Gateway 설치 확인

```
$ kubectl get gateway
NAME               AGE
bookinfo-gateway   43s
```

#### 3. Gateway YAML 파일 확인 

> samples/bookinfo/networking/bookinfo-gateway.yaml
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

이 gateway는 `istio=ingressgateway`인 svc에 설정을 추가한다(svc는 `istio-system` Namespace에 있다).

#### 4. svc에 Gateway가 적용된 것 확인

```
$ kubectl get svc -n istio-system -l istio=ingressgateway
NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   LoadBalancer   10.96.78.29   <pending>     15021:30564/TCP,80:32330/TCP,443:32761/TCP,31400:32742/TCP,15443:32720/TCP   147m
```

#### 5. 접속 테스트

`http://{ip}:{port}/productpage`경로로 접속해보자.

![](/STEP4-istio/images/web.png)


### 외부에서 Application Pod에 연결되는 프로세스

외부에서 Application Pod에 연결되는 프로세스를 살펴보면 다음과 같다. 외부 트래픽은 **Istio-Ingress**로부터 들어와서 **Gateway**를 통해 실제 서비스하는 **Application Pod**로 연결된다.


# Istio 대시보드: Kiali

`istioctl dashboar` 명령어를 사용하면 다양한 서비스에 접근이 가능하다. 원하는 서비스를 선택하면 된다.

```
$ istioctl dashboard
Access to Istio web UIs

Usage:
  istioctl dashboard [flags]
  istioctl dashboard [command]

Aliases:
  dashboard, dash, d

Available Commands:
  controlz    Open ControlZ web UI
  envoy       Open Envoy admin web UI
  grafana     Open Grafana web UI
  jaeger      Open Jaeger web UI
  kiali       Open Kiali web UI
  prometheus  Open Prometheus web UI
  zipkin      Open Zipkin web UI
```

**Kiali**를 사용하면 현재 구동되는 Application의 트래픽 구조를 확인할 수 있다. 


#### 1. Kiali 및 기타 add-on 설치

```
$ kubectl apply -f samples/addons
$ kubectl rollout status deployment/kiali -n istio-system
deployment "kiali" successfully rolled out
```

만약 설치 중에 오류가 발생하면 명령어를 다시 실행하면 된다. 


#### 2. Kiali svc serviceType 수정

Kiali의 svc이 serviceType이 ClusterIP이므로 NodePort로 변경해준다.

```
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
kiali                  ClusterIP      10.106.235.188   <none>        20001/TCP,9090/TCP                                                           19m
```

```
$ kubectl edit svc kiali -n istio-system
service/kiali edited
```

```
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
kiali                  NodePort       10.106.235.188   <none>        20001:31544/TCP,9090:30248/TCP                                               21m
```


#### 2. Kiali 브라우저 실행

```
$ istioctl dashboard kiali
http://localhost:20001/kiali
```

#### 3. Kiali 브라우저 접속

![](/STEP4-istio/images/kiali.png)


현재 트래픽이 거의 없는데, 일부러 트래픽을 발생시켜서 관찰해보자.

#### 4. 의도적인 트래픽 생성 및 관찰

아래 명령어를 통해 2초에 한번씩 트래픽을 발생시킨다.

```
$ watch "curl http://127.0.0.1:32330/productpage"
```

Kiali 홈 > [Graph] > [default Namespace]로 접속하면, 다음과 같이 트래픽이 그래프로 출력되는 것을 확인할 수 있다. 

![](/STEP4-istio/images/traffic-graph.png)

그외의 Application 들의 상태 조회 등 다양한 기능이 있지만, Kiali의 가장 큰 장점은 **트래픽이 지나가는 것을 확인**할 수 있다는 점이다.