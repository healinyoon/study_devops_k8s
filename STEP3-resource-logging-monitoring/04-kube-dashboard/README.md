# Kube Dashboard

### 개요

* Kubernetes cluster 범용 웹 기반 UI
* Cluster 자체 관리 가능
* Cluster에서 실행중인 응용 프로그램을 관리하고 문제 해결 가능
* https://github.com/kubernetes/dashboard

![](/STEP3-resource-logging-monitoring/images/04-kube-dashboard-ui.png)  
이미지 출처: https://github.com/kubernetes/dashboard

# Kubernetes Dashboard 설치

### 1. 설치 명령어 수행

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

출력에서 주목할 만한 Point는 `namespace/kubernetes-dashboard`이다. namespace에서 조회해보자.

### 2. 생성된 Resource 조회

```
$ kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-79c5968bdc-tv8d2   1/1     Running   0          2m47s
pod/kubernetes-dashboard-7448ffc97b-5wn28        1/1     Running   0          2m47s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.97.145.143    <none>        8000/TCP   2m47s
service/kubernetes-dashboard        ClusterIP   10.100.101.119   <none>        443/TCP    2m47s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2m47s
deployment.apps/kubernetes-dashboard        1/1     1            1           2m47s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-79c5968bdc   1         1         1       2m47s
replicaset.apps/kubernetes-dashboard-7448ffc97b        1         1         1       2m47s
```

`service/kubernetes-dashboard`의 type이 `ClusterIP`이므로, 외부에서 접속이 불가능한 상태이다. 외부 접속이 가능하도록, serviceType을 변경해주자.

### 3. kubernetes-dashboard ServiceType 변경

```
$ kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard
service/kubernetes-dashboard edited

아래와 같이 수정해준다.

수정 전)
  type: ClusterIP

수정 후)
  type: NodePort
```

다시 확인해보면, `ClusterIP -> NodePort` type으로 변경되었고 `443:31370/TCP`을 할당 받았다.

```
$ kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-79c5968bdc-tv8d2   1/1     Running   0          6m40s
pod/kubernetes-dashboard-7448ffc97b-5wn28        1/1     Running   0          6m40s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/dashboard-metrics-scraper   ClusterIP   10.97.145.143    <none>        8000/TCP        6m40s
service/kubernetes-dashboard        NodePort    10.100.101.119   <none>        443:31370/TCP   6m40s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           6m40s
deployment.apps/kubernetes-dashboard        1/1     1            1           6m40s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-79c5968bdc   1         1         1       6m40s
replicaset.apps/kubernetes-dashboard-7448ffc97b        1         1         1       6m40s
```

### 4. Kubernetes Dashboard Web 접속

https://{kubernetes cluster}:{node port}로 접속한다. 이때 https 인증이 되지 않으므로 [고급]에서 우회해서 가야한다(단, chrom, edge, safari에서는 우회 접속도 안되므로ㅠㅠ firefox를 사용하자..).

![](/STEP3-resource-logging-monitoring/images/04-kube-dashboard-1.png)  

접속 인증을 위해 token을 입력해야하는데, Service Account에 관련된 token을 가져오면 된다.

### 5. Service Account token 읽어오기

애플리케이션이 갖는 권한을 Service Account라고 한다(사용자가 갖는 권한은 User이다).

#### Step1. Service Account 조회

```
$ kubectl get sa -n kubernetes-dashboard
NAME                   SECRETS   AGE
default                1         22m
kubernetes-dashboard   1         22m
```

Serviec Account 구성이 궁금하다면 다음과 같이 조회해볼 수 있다.

```
$ kubectl get sa -n kubernetes-dashboard -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2021-01-11T09:17:29Z"
    name: default
    namespace: kubernetes-dashboard
    resourceVersion: "26499053"
    selfLink: /api/v1/namespaces/kubernetes-dashboard/serviceaccounts/default
    uid: fee7745b-eb9c-4029-9ced-2bb4a7ef7793
  secrets:
  - name: default-token-hgw8v
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kubernetes-dashboard"}}
    creationTimestamp: "2021-01-11T09:17:29Z"
    labels:
      k8s-app: kubernetes-dashboard
    managedFields:
    - apiVersion: v1
      fieldsType: FieldsV1
      fieldsV1:
        f:secrets:
          .: {}
          k:{"name":"kubernetes-dashboard-token-rrr5r"}:
            .: {}
            f:name: {}
      manager: kube-controller-manager
      operation: Update
      time: "2021-01-11T09:17:29Z"
    - apiVersion: v1
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
          f:labels:
            .: {}
            f:k8s-app: {}
      manager: kubectl-client-side-apply
      operation: Update
      time: "2021-01-11T09:17:29Z"
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
    resourceVersion: "26499052"
    selfLink: /api/v1/namespaces/kubernetes-dashboard/serviceaccounts/kubernetes-dashboard
    uid: ec365920-55e3-4401-8e78-f989254a3aa1
  secrets:
  - name: kubernetes-dashboard-token-rrr5r
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

#### Step2. Secret 조회

위의 Service Account는 Secret에 저장되어 있으므로, Secret을 조회해보자.

```
$ kubectl get secret -n kubernetes-dashboard
NAME                               TYPE                                  DATA   AGE
default-token-hgw8v                kubernetes.io/service-account-token   3      29m
kubernetes-dashboard-certs         Opaque                                0      29m
kubernetes-dashboard-csrf          Opaque                                1      29m
kubernetes-dashboard-key-holder    Opaque                                2      29m
kubernetes-dashboard-token-rrr5r   kubernetes.io/service-account-token   3      29m
```

#### Step3. Token 값 출력

위에서 얻은 Token 값을 출력해보자.

```
$ kubectl get secret -n kubernetes-dashboard kubernetes-dashboard-token-rrr5r -o yaml
(인코딩된 값이 출력된다.)
```

디코딩된 값을 얻기 위해서 아래 명령어를 입력한다.
```
$ kubectl describe secret -n kubernetes-dashboard kubernetes-dashboard-token-rrr5r
Name:         kubernetes-dashboard-token-rrr5r
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: ec365920-55e3-4401-8e78-f989254a3aa1

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkpjTllWSlE4dk1tNlZkcnpGRXNTS3hDQ2c1cHhCUzRNZkxfaUNiMUZyaXcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1ycnI1ciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVjMzY1OTIwLTU1ZTMtNDQwMS04ZTc4LWY5ODkyNTRhM2FhMSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.L-1jgEjLRY61FnYvyNTZ--r5nuZekp3smIA1gi_zcNbyOIDO2Qpt7E4x9EL28ziM7IsnKS2c6_bBeNNtFpVeHThXgztem96UtfVzOuRL_lZMfF-Ym5qhDxGO390cdm8I2ARzeBWjpdXKq28BFOYUqBoAjjhZHTpfbDApfAPRCwRvL_iVUV_aynV5DyDBQY9XGloRVklFDRgpNCSevGh5Jd7PxKpCkq9Q3wGqT3Lalx4e85r7qaYr3iu2wxznI5vSjgVzq9kQTHn4wbnNdq5pCs02Fdo_R4DIZ5ea-XUxIKfRmh7vCqXUOqfPwLB5H5HopuiiRPpur4QCn5VJsfwxiQ
```

### 6. Service Account token 으로 Web 로그인

위에서 얻은 token 값을 붙여넣기 하여, Kubernetes Dashboard에 로그인하자. **그런데 아무것도 표시되지 않는다.**

![](/STEP3-resource-logging-monitoring/images/04-kube-dashboard-2.png)  


이유는 Kubernetes를 Control할 권한이 해당 Serviec Account에 없기 때문이다 ㅠㅠ..

