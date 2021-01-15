# Kubernetes 보안을 위한 다양한 Resource

모든 통신은 TLS로!  
Kubernetes의 대부분의 access는 **Kube-apiserver**를 통하지 않고서는 불가능하다.

![](/STEP5-cluster-operation/images/04-security-resource-kube-apiserver.png)

### Access 가능한 User
* 사용자를 위한 User Account(= Static Token File)
    * File - User 아이디와 패스워드(token)
    * csv file을 만들어서 user 정보 저장
    * 가장 간단한 방법
    * kube-apiserver를 시작할 때 전달해주면, kube-apiserver가 file을 읽어들여서 아이디와 패스워드를 static하게 정해준다.
    * kube-apiserver가 시작할 때만 읽을 수 있기 때문에, kube-apiserver를 재시작해줘야 적용 가능하다.
* Service Accounts
    * Kubernetes 애플리케이션이 kube-apiserver와 통신(질의-응답 할 수 있는 권한)해야 하는 경우 사용
    * 예: Resource Monitoring, Multi Scheduling
* 인증서(Certificates)
* External Authentication Providers - LDAP 등

### 계정을 통해 무엇을 할 수 있는가?
* RBAC Authorization
* ABAC Authorization
* Node Authorization
* WebHook Mode