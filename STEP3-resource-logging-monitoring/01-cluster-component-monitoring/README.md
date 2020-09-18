# Kubernetes Monitoring System and Architecture

쿠버네티스 모니터링 시스템과 아키텍처를 살펴본다.

### Kubernetes monitoring tool 목적

* Kubernetes cluster 내의 애플리케이션 성능 검사
* Kubernetes는 리소스 사용량에 대한 상세 정보를 제공
* 애플리케이션의 성능을 평가하고 병목 현상을 제거하여 전체 성능 향상을 도모

# Kubernetes metric pipeline

### 1. Resource metric pipeline <- kuberentes가 지원하는 기본 파이프라인

* `kubectl top` 등의 유틸리티 관련된 metric들로 제한된 집합을 제공
* 단기 메모리 저장소인 `metric-server`에 의해 수집됨
* `metric-server`는 모든 Node를 발견하고 Kubelet에 CPU와 Memory를 질의
* Kubelet은 Kubelet에 포함된 cAdvisor를 통해 레거시 도커와 통합 후 => `metric-server` 리소스 메트릭으로 노출
* `/metrics/resource/v1beta1` API 사용

### 2. 완전한 metric pipeline
* 보다 풀부한 metric에 접근
* PV 등을 통해 메모리를 관리
* Cluster의 현재 상채를 기반으로 자동으로 스케일링하거나 Cluster를 조정
* CNCF 프로젝트인 "Prometheus"가 대표적
* `custom.metrics.k8s.io`, `external.metrics.k8s.io` API를 사용

# 전체 Architecture 구조

