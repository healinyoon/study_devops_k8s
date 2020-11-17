# Ingress-Nginx 개요
Ingress를 사용하면 L7의 웹 요청을 해석해서 단일 IP, 단일 port로 다수의 도메인과 서비스로 연결할 수 있다. 이 방법을 사용하면 웹페이지의 도메인을 같지만 다른 앱을 사용하는 것도 가능하다. 

하지만 쿠버네티스에서 기본적으로 지원하는 Ingress 오브젝트는 클라우드 환경에서만 사용할 수 있다. 클라우드에서 Ingress를 생성하면 외부에 게이트웨이를 생성하고 각 기능에 맞게 서비스를 연결한다. GCP의 경우에는 외부 게이트웨이에 L7 규칙이 적용되어있다. [HTTP(S) 부하분산용 GKE Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress?hl=ko)에 대한 내용을 참고하자.

![](/Ingress-Nginx/images/01-Ingress.jpeg)

여기서는 private 서버 환경에서 Ingress를 사용할 수 있는 Ingress-Nginx를 설치하고, Kubernetes pod 형태로 띄워서 설정하는 방법을 알아본다.

![](/Ingress-Nginx/images/02-Ingress-Nginx.png)  
그림 출처: https://www.nginx.com/products/nginx-ingress-controller/

Github 사이트에서 Ingress-Nginx에 대한 예제를 몇가지 제공하고 있다. 그중 가장 기본적인 예제를 분석하고 사용해보자.

* [Github/kunernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

# Ingress-Nginx 예제 분석

* [예제 파일 경로](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/provider/baremetal/deploy.yaml)

