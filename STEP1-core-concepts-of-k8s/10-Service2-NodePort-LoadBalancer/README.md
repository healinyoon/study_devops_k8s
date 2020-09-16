서비스를 외부에 노출하는 방법

# 서비스를 노출하는 3가지 방법
1) NodePort
    * Node의 자체 port를 사용하여 Pod로 리다이렉션
2) LoadBalancer
    * L4
    * 외부 게이트웨이를 사용하여 Node port로 리다이렉션
    * Cloud 환경에서 사용 가능
    * On-Premise 환경에서는 로드밸런스 리소스를 따로 두어야 하나??
3) Ingress
    * L7
    * 외부에 별도의 리소스를 만들어두고, 해당 리소스에 요청이 오면 Ingress 하는 방식
    * 하나의 IP로 다수의 서비스 제공 가능 

# NodePort

![](/STEP1-core-concepts-of-k8s/images/10-Service2-NodePort-LoadBalancer-k8s-archi.png)  

출처: https://en.wikipedia.org/wiki/Kubernetes

### NodePort 생성하기

Service yaml 파일 작성시 Service type을 NodePort로 지정해주면 된다. Port 범위는 30000-32767 만 가능하다.

* Service yaml 파일 예시
```
apiVersion v1
kind: Service
metadata:
  name: http-go-np
spec:
  type: NodePort
  ports:
  - port: 80            # Service Port
    targetPort: 8080    # Pod Port
    nodePort: 30001     # 최종적으로 서비스되는 Port
  selector:
    app: http-go
```

# LoadBalancer

* NodePort Service의 확장이다.
* 클라우드에서 사용 가능하다(클라우드를 사용하지 않는 경우, External DNS를 사용해야 가능).
* Service yaml 파일 작성시 Service type을 LoadBalancer로 지정해주면 된다.
* 외부에서 접근시 LoadBalancer의 IP 주소를 통해 액세스한다.

### LoadBalancer 생성하기

* Service yaml 파일 예시
```
apiVersion v1
kind: Service
metadata:
  name: http-go-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80            # Service Port = 최종적으로 서비스되는 Port
    targetPort: 8080    # Pod Port
  selector:
    app: http-go
```