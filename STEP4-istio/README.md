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