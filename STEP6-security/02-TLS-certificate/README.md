# TLS 인증서를 활용한 통신 이해

### SSL 통신 과정 이해
* Application 계층(7계층)인 HTTP와 Transfer 계층(4계층)인 TCP 사이에서 동작 = 즉 5~6 계층에서 동작
    * 5~6 계층은 SSL/TLS를 다루는데, SSL/TLS는 기존의 통신(HTTP, FTP, NNTP 등등..)을 지원하기 위해 만들어진 기술이다.
    * 데이터 암호화(기밀성), 데이터 무결성, 서버 인증 기능, 클라이언트 인증 기능 등을 지원한다.

![](/STEP6-security/images/02-TLS-certificate-1.png)

### 인증 순서

##### 1. Server가 CA 기관으로부터 인증서를 발급 받는다.

-- CA 기관에서 발급 --
인증서
공개 key = 자물쇠 = 암호화를 할 수 있지만, 해독이 안된다

-- Server --
개인 key = 열쇠 = 암호화를 할 수 있게 해준다

-- Client --
개인 key = 열쇠 = 암호화를 할 수 있게 해준다

##### 2. Client가 Server에 접근한다.
- Client hello

##### 3. Server가 Client에게 인증서와 공개키를 준다.
- 2~3 단계는 아직 암호화가 안된 상태이다.
- Server hello

##### 4. Client가 CA기관으로부터 안전한 인증서인지 질의하고 응답받는다.

##### 5. Client가 Server에게 Session 키를 준다.
    * Session 키: 공개키로 암호화된 키

##### 6. Server도 Client에게 Session 키를 준다.
이로써 쌍방 간 Session 키를 가지고 있게 되고, 이를 통해 통신한다.

![](/STEP6-security/images/02-TLS-certificate-2.png)
그림 출처: https://docs.pexip.com/admin/certificate_management.htm

### Kubernetes 인증서 위치
```
# ls -l /etc/kubernetes/pki/
total 60
-rw-r--r-- 1 root root 1245  1월 18 13:10 apiserver.crt
-rw-r--r-- 1 root root 1090  1월 18 13:10 apiserver-etcd-client.crt
-rw------- 1 root root 1679  1월 18 13:10 apiserver-etcd-client.key
-rw------- 1 root root 1679  1월 18 13:10 apiserver.key
-rw-r--r-- 1 root root 1099  1월 18 13:10 apiserver-kubelet-client.crt
-rw------- 1 root root 1679  1월 18 13:10 apiserver-kubelet-client.key
-rw------- 1 root root 1025  9월 16 15:39 ca.crt
-rw------- 1 root root 1679  9월 16 15:39 ca.key
drwxr-xr-x 2 root root 4096  9월 16 15:39 etcd
-rw------- 1 root root 1038  9월 16 15:39 front-proxy-ca.crt
-rw------- 1 root root 1675  9월 16 15:39 front-proxy-ca.key
-rw-r--r-- 1 root root 1058  1월 18 13:10 front-proxy-client.crt
-rw------- 1 root root 1679  1월 18 13:10 front-proxy-client.key
-rw------- 1 root root 1679  9월 16 15:39 sa.key
-rw------- 1 root root  451  9월 16 15:39 sa.pub

# ls -l /etc/kubernetes/pki/etcd/
total 32
-rw------- 1 root root 1017  9월 16 15:39 ca.crt
-rw------- 1 root root 1679  9월 16 15:39 ca.key
-rw-r--r-- 1 root root 1094  1월 18 13:10 healthcheck-client.crt
-rw------- 1 root root 1679  1월 18 13:10 healthcheck-client.key
-rw-r--r-- 1 root root 1155  1월 18 13:10 peer.crt
-rw------- 1 root root 1675  1월 18 13:10 peer.key
-rw-r--r-- 1 root root 1155  1월 18 13:10 server.crt
-rw------- 1 root root 1679  1월 18 13:10 server.key
```

* `client`가 붙은 것은 client용, 그렇지 않은 것은 server용으로 보면 된다.
* `.crt`: certificate
* `.key`: 개인 키
* `ca.crt`, `ca.key`: CA 기관용이 별도로 있다.

### 정확한 TLS 인증서 사용 점검 필요

* 적절한 키를 사용하는지 확인하려면 manifests 파일에서 실행하는 certificate 확인이 필요하다.
* 적절한 키를 사용하지 않으면 에러가 발생한다.

```
# ls -l /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2125  1월 18 13:10 etcd.yaml
-rw------- 1 root root 3828  1월 18 13:10 kube-apiserver.yaml
-rw------- 1 root root 3350  1월 18 13:10 kube-controller-manager.yaml
-rw------- 1 root root 1384  1월 18 13:10 kube-scheduler.yaml
```

* kubelet의 pki 위치는 다르다.

```
# ls -l /var/lib/kubelet/pki/
total 12
-rw------- 1 root root 1082  9월 16 15:39 kubelet-client-2020-09-16-15-39-31.pem
lrwxrwxrwx 1 root root   59  9월 16 15:39 kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2020-09-16-15-39-31.pem
-rw-r--r-- 1 root root 2225  9월 16 15:39 kubelet.crt
-rw------- 1 root root 1679  9월 16 15:39 kubelet.key

# ls -l /var/lib/kubelet/
total 36
-rw-r--r-- 1 root root  862  1월 18 13:20 config.yaml
-rw------- 1 root root   62  9월 16 15:39 cpu_manager_state
drwxr-xr-x 2 root root 4096  1월 18 13:21 device-plugins
-rw-r--r-- 1 root root  165  1월 18 12:55 kubeadm-flags.env
drwxr-xr-x 2 root root 4096  9월 16 15:39 pki
drwxr-x--- 2 root root 4096  9월 16 15:39 plugins
drwxr-x--- 2 root root 4096  9월 16 15:39 plugins_registry
drwxr-x--- 2 root root 4096  1월 18 13:21 pod-resources
drwxr-x--- 8 root root 4096  1월 18 13:19 pods

# cat /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: cgroupfs
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

# TLS 인증서 정보 확인과 자동 갱신 방법

### 인증서 정보 확인하기

#### 인증서 정보 조회 명령어: 
```
# openssl x509 -in {인증서 경로} -text

# openssl x509 -in apiserver-etcd-client.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 8386199189358326558 (0x7461c3bb115f331e)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = etcd-ca
        Validity
            Not Before: Sep  8 07:34:55 2020 GMT
            Not After : Jan 14 09:29:10 2022 GMT
        Subject: O = system:masters, CN = kube-apiserver-etcd-client
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:ea:70:7a:a2:26:fd:de:cd:4a:43:bc:e2:ec:b1:
```

* Issuer: 인증서 발행 주체
    * CN은 common names를 의미
* Subject: 인증서 발행 대상
* Validity: 사용 가능 기간
    * Not Before ~ Not After 사이의 기간동안 사용 가능 
    * Kubernetes 인증서는 기본적으로 1년
    * CA 인증서는 10년
* Public-key: 공개키

#### 조금 특별한 `ca.cert` 파일 살펴보기:

* Issuer과 Subject가 **kubernetes** 이다.
* 인증 기간은 10년이다.

```
# openssl x509 -in ca.crt  -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Sep  8 07:34:53 2020 GMT
            Not After : Sep  6 07:34:53 2030 GMT
        Subject: CN = kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:af:93:95:73:bf:21:94:e2:9f:b1:37:de:b5:d4:
```

### 인증서 목록

`.conf`는 파일 내부에 인증서가 저장되어 있는 경우이다.

![](/STEP6-security/images/02-TLS-certificate-3.png)
 
### CA 목록

![](/STEP6-security/images/02-TLS-certificate-4.png)

### 모든 인증서 갱신하기

[쿠버네티스 공식 문서](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

#### Check certificate expiration

* 인증서 남은 기간을 확인하는 방법
* 명령어: `kubeadm certs check-expiration`

```
# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jan 14, 2022 09:29 UTC   356d                                    no
apiserver                  Jan 14, 2022 09:29 UTC   356d            ca                      no
apiserver-etcd-client      Jan 14, 2022 09:29 UTC   356d            etcd-ca                 no
apiserver-kubelet-client   Jan 14, 2022 09:29 UTC   356d            ca                      no
controller-manager.conf    Jan 14, 2022 09:29 UTC   356d                                    no
etcd-healthcheck-client    Jan 14, 2022 09:28 UTC   356d            etcd-ca                 no
etcd-peer                  Jan 14, 2022 09:28 UTC   356d            etcd-ca                 no
etcd-server                Jan 14, 2022 09:28 UTC   356d            etcd-ca                 no
front-proxy-client         Jan 14, 2022 09:29 UTC   356d            front-proxy-ca          no
scheduler.conf             Jan 14, 2022 09:29 UTC   356d                                    no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Sep 06, 2030 07:34 UTC   9y              no
etcd-ca                 Sep 06, 2030 07:34 UTC   9y              no
front-proxy-ca          Sep 06, 2030 07:34 UTC   9y              no
```

#### Muanual certificate renewal
* 인증서를 renewal 한다.
* 명령어: `kubeadm certs renew all`

```
# kubeadm certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.
```

#### Automatic certificate renewal
* kubeadm은 컨트롤 플레인을 업그레이드하면 모든 인증서를 자동으로 갱신한다.

# TLS 인증서를 활용한 유저 생성하기

### Step 1. CA를 사용하여 직접 CSR 승인하기(1인 2역: Client와 CA 기관)

CSR(certificate signing request)이란: 인증서 서명 요청이란 뜻으로, 인증서 발급을 위한 필요한 정보를 담고 이는 인증서 신청 형식의 데이터이다. CSR에 포함되는 내용으로는 개인 키 생성 단계에서 만들어지는 개인 키(private key)와 공개 키(public key)의 키 쌍 중에서 공개 키가 포함되며, 인증서가 적용되는 도메인에 대한 정보 등이 있다.

> (참고) 키 발급 순서
1. 개인 키를 생성한다.
2. 개인키로 CSR을 만들고 CA에게 보낸다.
3. CA는 CSR을 보고 CRT를 발급해준다(이후부터는 CSR이 필요없어진다).

출처: https://m.blog.naver.com/PostView.nhn?blogId=swh0506&logNo=30186498871&proxyReferer=https:%2F%2Fwww.google.com%2F

#### 1) 개인 키 생성하기
* 길이 2048만큼의 개인 키 생성

```
$ openssl genrsa -out ringu.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
............+++++
............................................+++++
e is 65537 (0x010001)
```

#### 2) 개인 키를 사용하여 CSR 생성하기
* CN: 사용자 이름
* O: 그룹 이름

``` 
$ openssl req -new -key ringu.key -out ringu.csr -subj "/CN=ringu/O=k8sproject"
```

명령어를 통해 `ringu.csr` 을 output으로 받는다. 이 CSR을 활용하여 CA에게 인증을 요청할 수 있다.


다음과 같은 오류가 발생할 경우: 
```
$ openssl req -new -key ringu.key -out ringu.csr -subj "/CN=ringu/O=k8sproject"
Can't load /home/ringu/.rnd into RNG
140443701965248:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/ringu/.rnd
```
아래 명령어 실행 후 다시 시도한다.
```
$ echo password > .rnd
```

생성된 개인 키와 CSR을 확인해보자.
```
$ ls -al ringu*
-rw-rw-r-- 1 ldccai ldccai  915 Jan 25 04:18 ringu.csr
-rw------- 1 ldccai ldccai 1675 Jan 25 04:17 ringu.key
```

### 3) CSR을 CA 기관에 전달하여 승인받고, CRT 생성 받기
> 원래는 CSR을 보내서 CRT을 발급받는 CA기관이 별도로 존재하지만, 지금 예시는 1인 2역 중이므로 스스로가 CA 기관이라고 생각하고 직접 승인해준다.

먼저 CSR의 정보를 조회해보자. 아직 CA 기관의 인증(sign)을 받지 않았기 때문에, 아직 인증 받았다는 정보가 포함되어 있지 않다.
```
$ openssl req -in ringu.csr -text
```

Kubernetes 클러스터 인증 기관(CA)이 요청을 승인하도록 해보자. 내부에서 직접 승인하는 경우 `pki` 디렉토리에 있는 `ca.key`와 `ca.crt`를 사용하여 승인 가능하다. 아래 명령어는 `ringu.csr`을 승인하여 최종 인증서인 `ringu.crt`를 생성한다. 

```
$ sudo openssl x509 -req -in ringu.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ringu.crt -days 365
Signature ok
subject=CN = ringu, O = k8sproject
Getting CA Private Key
```

`-days` 옵션을 사용하면 며칠간 인증서가 유효한지 설정할 수 있다.

생성된 CRT을 확인해보자.
```
$ ls -al ringu*
-rw-rw-r-- 1 ldccai ldccai 1017 Jan 25 04:27 ringu.crt
-rw-rw-r-- 1 ldccai ldccai  915 Jan 25 04:18 ringu.csr
-rw------- 1 ldccai ldccai 1675 Jan 25 04:17 ringu.key
```

이제 CSR을 필요없으므로 제거해도 된다.
```
$ rm -rf ringu.csr
```

### Step 2. 인증 사용을 위해 쿠버네티스에 CRT를 등록

CRT를 사용할 수 있도록 `kubectl`을 사용하여 등록하자. 

#### 1) 가장 먼저 사용자를 등록하고,

```
$ kubectl config set-credentials ringu --client-certificate=ringu.crt --client-key=ringu.key
User "ringu" set.
```

#### 2) 생성한 사용자와 cluster를 연결해준다.
```
$ kubectl config set-context ringu@kubernetes --cluster=kubernetes --namespace=office --user=ringu
Context "ringu@kubernetes" created.
```

일반적으로 `kubectl config set-context {context name}`에서 context name 형식을 `{사용자 명}@{도메인 정보(--cluster의 값과 일치)}`으로 한다.

#### 3) 생성한 계정으로 로그인
```
$ kubectl config use-context ringu@kubernetes
Switched to context "ringu@kubernetes".
```

로그인 한 다음부터는 `ringu`의 권한을 가지고 행동하게 된다.
 
#### 4) 다음 명령을 사용하여 사용자 권한으로 실행 가능

현재는 사용자에게 권한을 할당하지 않으면 실행되지 않는다.

```
$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "ringu" cannot list resource "pods" in API group "" in the namespace "office"
```

#### 5) 다시 관리자 권한으로 돌아가자.

```
$ kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```


# 연습 문제

### 문제
dev1 팀에 john이 참여 했다. John을 위한 인증서를 만들고 승인해보자.


### 풀이

##### 1. 개인 key 생성
```
$ openssl genrsa -out john.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.........................+++++
...........+++++
e is 65537 (0x010001)
```

##### 2. CSR 생성
```
$ openssl req -new -key john.key -out john.csr -subj "/CN=john/O=k8sproject"
$ ls -al john*
-rw-rw-r-- 1 ldccai ldccai  911 Jan 25 06:15 john.csr
-rw------- 1 ldccai ldccai 1679 Jan 25 06:14 john.key
```

##### 3. CRT 발급
```
$ sudo openssl x509 -req -in john.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out john.crt -days 365
Signature ok
subject=CN = john, O = k8sproject
Getting CA Private Key

$ ls -al john*
-rw-r--r-- 1 root   root   1013 Jan 25 06:15 john.crt
-rw-rw-r-- 1 ldccai ldccai  911 Jan 25 06:15 john.csr
-rw------- 1 ldccai ldccai 1679 Jan 25 06:14 john.key
```

##### 4. User 생성
```
$ kubectl config set-credentials john --client-certificate=john.crt --client-key=john.key
User "john" set.
```

##### 5. Context 생성(Cluster와 User 이어주기)
```
$ kubectl config set-context john@kubernetes --cluster=kubernetes --namespace=dev1 --user=john
Context "john@kubernetes" created.
```

##### 6. 생성한 Context로 로그인
```
$ kubectl config use-context john@kubernetes
Switched to context "john@kubernetes".
```

##### 7. 생성한 Context 테스트
```
$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "john" cannot list resource "pods" in API group "" in the namespace "dev1"
```