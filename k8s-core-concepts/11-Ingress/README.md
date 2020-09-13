# Ingress 란

 Kubernetes의 Ingress란 HTTP(S) 기반의 L7 로드밸런싱 기능을 제공하는 컴포넌트이다. 외부에서 쿠버네티스 클러스터 내부로 들어오는 네트워크 요청(= Ingress 트래픽)을 어떻게 처리할지 정의한다. 다시 말하자면, Ingress는 외부에서 Kubernetes에서 실행 중인 Deployment와 Service에 접근하기 위한, 일종의 관문(Gateway) 같은 역할을 담당한다.
