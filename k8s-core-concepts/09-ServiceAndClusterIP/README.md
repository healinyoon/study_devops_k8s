# Services의 등장 배경

### Pod의 문제점

* Pod는 일시적으로 생성한 컨테이너의 집합
  * Pod가 지속적으로 생겨났을 때 서비스하기 적합하지 않음
  * Pod의 생성, 삭제, 이전 등에 따른 관리의 어려움
* Pod는 IP 주소가 변동되기 때문에 로드밸런싱을 관리해줄 또 다른 리소스가 필요

### Services의 역할

1) 로드 밸런싱
2) Pod 선택

# Service 생성 방법

### 방법 1) expose 명령어 사용

`kubectl expose`로 생성(가장 쉬운 방법)  
```
$ kubectl expose deployment {Deployement 명} --type=LoadBalancer --name=my-servic
```

### 방법 2) yaml 파일로 버전 관리 가능

yaml 파일 생성
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp          <-- Label이 MyApp인 Pod에 로드 밸런싱을 해준다
  ports:
  - protocol: TCP     
    port: 80          <-- 외부 접속 port
    targetPort: 9376  <-- Pod 오픈 port
```

yaml 파일 실행
```
$ kubectl create -f {Service yaml}
```

생성된 Service 확인
```
$ kubectl get svc
```

# Service 세부 기능

### Service 로드밸런싱 기능 확인

* Service를 생성하면 EXTERNAL -IP를 아직 받지 못한 것을 확인
* `kubectl exec {Pod 명} --curl` 명령어로 확인

### Pod간 통신을 위한 ClusterIP

* 가장 기본인 Service의 타입
* 외부에는 노출되지 않고 내부에서만 사용하기 위한 것

### Service 세션 고정하기

* Service가 다수의 Pod로 구성하면 웹 서비스의 세션이 유지 되지 않음
  * 사용자의 기록을 저장되지 않음
  * 이를 위해 처음 접속했던 ClientIP를 그대로 유지해주는 방법이 필요(동일한 Pod로 접속하도록)

**=> sessionAffinity: ClientIP 옵션 사용**

yaml 파일 예시)
```
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  sessionAffinity: ClientIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: http-go
```

### 다중 Ports 서비스 방법

서버에 여러 Port를 한꺼번에 지원하고 싶을 때 사용
* Port를 그대로 나열해서 사용
* rediraction 방식

yaml 파일 예시)
```
spec:
  sessionAffinity: ClientIP
  ports:            <-- 's' 가 붙는 속성은 '-' 기호를 사용하여 배열을 나열할 수 있다.
  - name: http      <-- '-' 기호는 다음 '-'가 나올 때까지가 1 set이 된다.
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
```

**Tip!**
's'가 붙는 속성은 yaml 파일의 리스트(배열) 형식  
ports 외에도 containers 등 다양한 속성이 복수개의 요소를 가짐


### Service하는 IP 정보 확인

* 서비스 세부 사항에는 연결된 IP 정보가 있음  
* `describe` 명령어로 조회

```
$ kubectl describe svc {Service 명}

...
Port:       http 80/TCP
TargetPort: 8080/TCP
EndPoints:  x.x.x.x:xxxx, x.x.x.x:xxxx
...
```

### 외부 IP 연결 설정하기(yaml)

* Service와 Endpoints 리소스 모두 생성 필요
* 외부에 로드밸런싱할 수 있도록 함(내부 -> 외부로 나갈 때)

* external-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
```

* external-endpoint.yaml
```
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service
subsets:
- addresses:
  - ip: 11.11.11.11
  - ip: 22.22.22.22
  ports:
  - port: 80
```
 
* Service와 Endpoints 연결 구조
[](/images/9-ServiceAndClusterIP-externalIP.jpeg)

그림 출처: 인프런-DevOps를 위한 쿠버네티스마스터: 서비스와 ClusterIP 소개
