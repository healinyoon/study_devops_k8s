# RBAC를 활용한 Role 기반 액세스 컨트롤

[쿠버네티스 RBAC 공식 문서](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

이전 내용까지는 사용자를 생성하고 사용하는 방법을 정리하였다. 이제 생성한 사용자에게 권한을 부여하는 방법을 살펴본다.

### 역할 기반 액세스 제어(RBAC: Role Based Access Control) 개요

* 기업 내에서 개발 사용자의 역할을 기반으로 컴퓨터, 네트워크 리소스에 대한 액세스를 제어한다.
* `rbac.authorization.k8s.io` API를 사용하여 정의한다.
* 권한 결정을 내리고, 관리자가 Kubernetes API를 통해 정책을 동적으로 구성한다.
* RBAC를 사용하여 role을 정하려면 apiserver에 `--authorization-mode=RBAC` 옵션이 필요하다(kubernetes 기본 값으로 설정되어 있다).


### rbac.authorication.k8s.io API

RBAC를 다루는 `rbac.authorication.k8s.io` API는 총 4가지의 리소스를 컨트롤한다.

1. Role
2. RoleBinding
3. ClusterRole
4. ClusterRoleBinding

![](/STEP6-security/images/04-RBAC-1.png)

Role은 Namespace 내부에 종속적인 반면에 ClusterRole은 Cluster 전체에서 사용 가능하다. ClusterRole의 권한이 더 막강하기 때문에 일반적으로 Role은 일반 사용자에게, ClusterRole은 관리자에게 할당된다.

### Role과 RoleBinding의 차이

#### Role

* "누가" 하는 것인지는 정하지 않고 Role만 정의한다.
* 일반 Role의 경우에는 Namespace 단위로 역할을 관리한다.
* ClusterRole은 Namespace에 종속적이지 않고, 전체 Cluster에서 역할을 관리한다.

#### RoleBinding

* Role을 정의하는 대신에 참조할 Role을 정의한다(roleRef).
* 어떤 사용자에게 어떤 권한을 부여하는지 정하는 Binding 리소스이다.
* Role에는 RoleBinding ClusterRole에는 ClusterRoleBinding이 필요하다.

![](/STEP6-security/images/04-RBAC-2.png)

### Role 작성 요령

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: office
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Role에는 apiGroups, resource, 작업 가능한 동작 등을 작성하게 된다. 작성 가능한 API의 종류는 공식 문서에서 확인할 수 있다.

* https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20 
* https://kubernetes.io/docs/reference/access-authn-authz/rbac

### Role Binding: 사용자 권한 할당

* RoleBinding을 사용하여 `office` Namespace에 대한 권한을 할당한다. // ClusterRoleBinding은 Namespace 설정 부분이 없다.
* `kubectl create -f` 명령어를 사용한다.

```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "john" to read pods in the "office" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: office
subjects:                                   # 여러개의 subject를 입력 가능하다
- kind: User
  name: john # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:                                    # 참조하려는 Role 또는 ClusterRole에 대한 정보를 입력한다
  kind: Role                                # Role or ClusterRole
  name: pod-reader                          # Binding 하려는 Role(또는 ClusterRole)의 이름과 일치시켜줘야 한다
  apiGroup: rbac.authorization.k8s.io
```

### 사용자 권한 테스트

아래의 명령어로 사용자의 권한을 테스트 해보자.

#### 성공해야 하는 명령어

```
$ kubectl --context=ringu-context get pods -n office
$ kubectl --context=ringu-context run --generator=pod-run/v1 nginx --image=nginx -n office
```

#### 실패해야 하는 명령어

```
$ kubectl --context=ringu-context get pods
$ kubectl --context=ringu-context run --generator=pod-run/v1 nginx --image=nginx
```

### 권한의 종류

[쿠버네티스 공식 문서](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

![](/STEP6-security/images/04-RBAC-3.png)
