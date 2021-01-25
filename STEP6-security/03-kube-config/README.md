# Kube Config 파일을 사용한 인증 개요

### (Kube Config 파일을 사용하지 않고도) 직접 curl을 사용하여 요청 가능하다.

* key, cert, cacert 키를 가지고 직접 요청 가능하다.
* **그러나 매번 이 요청을 사용하기에는 무리가 있다.**

```
$ curl https://kube-api-server:6443/api/v1/pods\
--key user.key
--cert user.crt
--cacert ca.crt
```

### Kube Config 파일을 사용한 인증

위의 curl을 사용하는 방법은 불편하므로, Kube Config View에 적용해서 사용한다.

#### 명령어:
```
$ kubectl config view --kube config={config file}
```

#### Kube Config 파일 살펴보기

```
$ kubectl config view
```

Kube Config Veiws는 크게 **clusters**, **contexts**, **users** 3가지 부분으로 나누어져 있다. 가장 먼저 **clusters**에 대한 출력을 볼 수 있다. 현재는 한 개의 cluster만 생성하여 사용하고 있기 때문에 한 개의 cluster만 출력된다.
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.1.11.7:6443
  name: kubernetes
```

다음으로 **users**에 대한 출력을 살펴보자. 각 사용자 계정의 인증서 경로 또는 token이 표시되고 있다. `kubernetes-admin`의 경우 `REDACTED`로 표시되는데 `~/.kube/config` 파일에 인증 값이 String 타입으로 저장되어 있다는 뜻이다.
```
 users:
- name: john
  user:
    client-certificate: /home/ldccai/john.crt
    client-key: /home/ldccai/john.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: ringu
  user:
    client-certificate: /home/ldccai/.certs/ringu.crt
    client-key: /home/ldccai/.certs/ringu.key
- name: user01
  user:
    token: REDACTED
```

마지막으로 **contexts**는 cluster와 useres를 조합해서 생성한 것이다. cluster와 user의 조합을 통해 생성된 context name와 namespace 등의 정보가 각각 표시되고 있다.
```
contexts:
- context:
    cluster: kubernetes
    namespace: dev1
    user: john
  name: john@kubernetes
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
- context:
    cluster: kubernetes
    namespace: office
    user: ringu
  name: ringu@kubernetes
- context:
    cluster: kubernetes
    namespace: frontend
    user: user01
  name: user01-context
```

 추가로, 중간에 **current-context** 정보가 포함되어 있는데 이는 `use context`로 선택한 현재 context가 등록되어 있다. 


# Kube Config의 구성

`~/.kube/config` 파일을 확인하여 Kube Config의 구성을 자세히 살펴보자.

위의 설명과 동일하게 파일은 세 가지 부분으로 작성되어 있다.

* clusters: 연결할 쿠버네티스 클러스터의 정보 입력
* users: 사용할 권한을 가진 사용자 입력
* context: cluster와 user를 함께 입력하여 권한 할당

 ![](/STEP6-security/images/03-kube-config.png)

추가로 `kubernetes-admin`의 인증 값이 String 타입으로 저장되어 있는 것을 확인할 수 있다.


### 인증 사용자 바꾸는 명령어
```
$ kubectl config use-context john@kubernetes
```

또는 사용자를 변경하지 않고도 옵션으로 바로 적용해볼 수 있다.
```
$ kubectl get pod --context john@kubernetes

$ kubectl get pod --user john

$ kubectl get pod --as john
```

현재까지는 forbidden이 나오면 성공이다.


