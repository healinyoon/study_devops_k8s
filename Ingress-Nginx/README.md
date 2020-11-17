# Ingress-Nginx 개요
Ingress를 사용하면 L7의 웹 요청을 해석해서 단일 IP, 단일 port로 다수의 도메인과 서비스로 연결할 수 있다. 이 방법을 사용하면 웹페이지의 도메인을 같지만 다른 앱을 사용하는 것도 가능하다. 

하지만 쿠버네티스에서 기본적으로 지원하는 Ingress 오브젝트는 클라우드 환경에서만 사용할 수 있다. 클라우드에서 Ingress를 생성하면 외부에 게이트웨이를 생성하고 각 기능에 맞게 서비스를 연결한다. GCP의 경우에는 외부 게이트웨이에 L7 규칙이 적용되어있다. [HTTP(S) 부하분산용 GKE Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress?hl=ko)에 대한 내용을 참고하자.

![](/Ingress-Nginx/images/01-Ingress.jpeg)

여기서는 private 서버 환경에서 Ingress를 사용할 수 있는 Ingress-Nginx를 설치하고, Kubernetes pod 형태로 띄워서 설정하는 방법을 알아본다.

![](/Ingress-Nginx/images/02-Ingress-Nginx.png)  
그림 출처: https://www.nginx.com/products/nginx-ingress-controller/

Github 사이트에서 Ingress-Nginx에 대한 예제를 몇가지 제공하고 있다. 그중 가장 기본적인 예제를 분석하고 사용해보자.

* [github/kunernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

# Ingress-Nginx 예제 분석

* [예제 파일 경로](https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/provider/baremetal/deploy.yaml)

예제가 매우 길지만, 이러한 예제를 분석할 수 있어야 우리 환경에서 Ingress-Nginx가 정확히 어떻게 동작할지 예상할 수 있다. 라인 수는 거의 650 라인 정도 된다. 하나씩 살펴보고 어떤 기능을 가지는 지 알아보자.

# http-go 서비스 설치
Ingress-Nginx를 설치하기 앞서, 테스트에 사용할 `http-go` 웹 서비스를 하나 띄우고 애플리케이션이 잘 동작하는지 확인하자. `http-go`는 go로 작성되었으며, 8080 포트로 웹 서비스를 하는 간단한 이미지이다. 네트워크는 NodePort로 열어준다.

```
$ kubectl create deployment http-go --image=gasbugs/http-go
deployment.apps/http-go created

$ kubectl expose deployment http-go --port=8080 --type=NodePort
service/http-go exposed
```

`http-go` 애플리케이션이 정상 동작하는지 확인하자.

1) 포트를 확인하고
```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
http-go      NodePort    10.96.162.86   <none>        8080:30268/TCP   3m54s
```

2) IP를 확인하고
```
$ kubectl get nodes -o wide
NAME      STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master    Ready    master   70d   v1.19.0   10.1.11.7     <none>        Ubuntu 18.04.5 LTS   5.4.0-1023-azure   docker://19.3.12
worker1   Ready    <none>   70d   v1.19.0   10.1.11.8     <none>        Ubuntu 18.04.5 LTS   5.4.0-1023-azure   docker://19.3.12
worker2   Ready    <none>   70d   v1.19.0   10.1.11.9     <none>        Ubuntu 18.04.5 LTS   5.4.0-1025-azure   docker://19.3.12
```

3) 동작을 테스트 한다.
```
$ curl -i http://10.1.11.7:30268
HTTP/1.1 200 OK
Date: Tue, 17 Nov 2020 09:01:32 GMT
Content-Length: 33
Content-Type: text/plain; charset=utf-8

Welcome! http-go-568f649bb-r6psq

$ curl -i http://10.1.11.8:30268
HTTP/1.1 200 OK
Date: Tue, 17 Nov 2020 09:01:38 GMT
Content-Length: 33
Content-Type: text/plain; charset=utf-8

Welcome! http-go-568f649bb-r6psq

$ curl -i http://10.1.11.9:30268
HTTP/1.1 200 OK
Date: Tue, 17 Nov 2020 09:01:44 GMT
Content-Length: 33
Content-Type: text/plain; charset=utf-8

Welcome! http-go-568f649bb-r6psq
```

# Ingress-Nginx 적용
이제 Ingress-Nginx를 직접 Kubernetes에 설치하고 Ingress가 정상적으로 동작하는지 테스트해보자. 

### Ingress-Nginx 설치
* [Ingress-Nginx Installation Guide](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

위의 경로에서 본인의 환경에 해당하는 명령어를 사용하면 된다.
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
```

위에서 중요한 것은 service 오브젝트이다.
```
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
```

`deployement`를 생성하고, 해당 deployment를 열어줄 `service`를 생성한 것으로, 아래에 해당하는 부분이다.

![](/Ingress-Nginx/images/03-Ingress-Nginx-Service.png)  

service의 내용을 확인해보자. `ingress-nginx` Namespace에 설치되어 있는 것을 볼 수 있다.
```
$ kubectl get service/ingress-nginx-controller -n ingress-nginx
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.98.119.197   <pending>     80:32194/TCP,443:30483/TCP   7m50s
```

이제 `ingress-nginx`와 `http-go`를 연결하는 ingress를 만들어주면 된다.

### Ingress 룰 생성
다음 그림처럼 룰을 만들어야 외부에서 요청이 왔을 때, ingress-nginx가 http-go로 포워딩해줄 수 있다. Ingress 룰은 매우 단순하다. 도메인 경로로 규칙을 지정하는데, 여기서는 `gasbugs.com/hostname`으로 요청이 들어오면 `http-go`로 연결해준다.

![](/Ingress-Nginx/images/04-Ingress-Rule.png)  

내용을 작성할 때는 다음을 주의해야 한다.
* `http-go`와 동일한 네임스페이스에 작성해야 한다.
* 앞서 생성한 Service 이름이 `serviceName` 오브젝트와 동일해야 한다.
* `servicePort` 오브젝트는 Service가 동작하는 포트를 의미해야 한다(Service의 포트와 Pod의 포트가 다른 경우에도 Service의 Pod를 적는다).

추가로, 도메인 이름은 gasbugs.com을 사용하고 /hostsname 경로를 사용하도록 만들었기 때문에 반드시http://gasbugs.com/hostsname으로 요청해야만 http-go로 연결된다.



ingrees-nginx 설치부터 얘로 다시 해보자
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created

$ kubectl get svc ingress-nginx-controller -n ingress-nginx
NAME                       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   NodePort   10.104.233.162   <none>        80:30424/TCP,443:31648/TCP   24s
```

여기서부터 다시해보기