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
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Role에는 apiGroups, resource, 작업 가능한 동작 등을 작성하게 된다. 작성 가능한 API의 종류는 공식 문서에서 확인할 수 있다. 참고로 apiGroups가 비어 있는 경우는 `core` API Groups을 의미한다. 

* https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20 
* https://kubernetes.io/docs/reference/access-authn-authz/rbac

### Role Binding: 사용자 권한 할당

Role을 생성한 다음에는 RoleBinding을 생성하여 `ringu` 사용자에게 `default` Namespace에 대한 권한을 할당한다(참고로 ClusterRoleBinding은 Namespace 설정 부분이 없다).

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:                                   # 여러개의 subject를 입력 가능하다
- kind: User                                # 권한을 받을 대상의 정보를 입력한다(ServiceAccounts라면 ServiceAccounts를 입력)
  name: ringu
  apiGroup: rbac.authorization.k8s.io
roleRef:                                    # 참조하려는 Role or ClusterRole에 대한 정보를 입력한다
  kind: Role                                # Role or ClusterRole
  name: pod-reader                          # Binding 하려는 Role(or ClusterRole)의 이름과 일치시켜줘야 한다
  apiGroup: rbac.authorization.k8s.io
```

### 사용자 권한 테스트

아래의 명령어로 사용자의 권한을 테스트 해볼 수 있다.

#### 성공해야 하는 명령어

```
$ kubectl --context=ringu@kubernetes get pods -n default
$ kubectl --context=ringu@kubernetes get pods -n default -w
```

#### 실패해야 하는 명령어

```
$ kubectl --context=ringu@kubernetes get pods -n kube-system
$ kubectl --context=ringu@kubernetes get pods -n kube-system -w
```

### 권한의 종류

[쿠버네티스 공식 문서](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

![](/STEP6-security/images/04-RBAC-3.png)


# RBAC 실습 1 - Role과 RoleBinding 생성하기

### 1. Role 생성

> pod-reader-role.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```
$ kubectl create -f pod-reader-role.yaml
role.rbac.authorization.k8s.io/pod-reader created
```

### 2. RoleBinding 생성

> pod-reader-role-binding.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: ringu
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
$ kubectl create -f pod-reader-role-binding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
```

### 3. 사용자 권한 테스트

* get 권한 확인
```
$ kubectl get pod --context=ringu@kubernetes -n default
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79c697d759-85bfd       2/2     Running   0          12d
http-go-7fdf786dff-jmvnb          2/2     Running   0          10d
productpage-v1-65576bb7bf-psj4k   2/2     Running   0          10d
ratings-v1-7d99676f7f-xnzz8       2/2     Running   0          10d
reviews-v1-987d495c-r52wh         2/2     Running   0          12d
reviews-v2-6c5bf657cf-bkkqw       2/2     Running   0          10d
reviews-v3-5f7b9f4f77-2fpzb       2/2     Running   0          12d
```

* watch 모드 확인
```
$ kubectl get pod --context=ringu@kubernetes -n default -w
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79c697d759-85bfd       2/2     Running   0          12d
http-go-7fdf786dff-jmvnb          2/2     Running   0          10d
productpage-v1-65576bb7bf-psj4k   2/2     Running   0          10d
ratings-v1-7d99676f7f-xnzz8       2/2     Running   0          10d
reviews-v1-987d495c-r52wh         2/2     Running   0          12d
reviews-v2-6c5bf657cf-bkkqw       2/2     Running   0          10d
reviews-v3-5f7b9f4f77-2fpzb       2/2     Running   0          12d
```

# RBAC 실습 2 - ClusterRole 확인해보기

### 1. admin ClusterRole 조회

```
$ kubectl get clusterrole | grep admin
admin                                                                  2020-09-08T07:35:34Z
cluster-admin                                                          2020-09-08T07:35:34Z
system:aggregate-to-admin                                              2020-09-08T07:35:34Z
system:kubelet-api-admin                                               2020-09-08T07:35:34Z
```

`cluster-admin` ClusterRole의 설정을 조회해보자.

### 2. cluster-admin ClusterRole 설정 조회

```
$ kubectl get clusterrole cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-09-08T07:35:34Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:rbac.authorization.kubernetes.io/autoupdate: {}
        f:labels:
          .: {}
          f:kubernetes.io/bootstrapping: {}
      f:rules: {}
    manager: kube-apiserver
    operation: Update
    time: "2020-09-08T07:35:34Z"
  name: cluster-admin
  resourceVersion: "44"
  uid: 54de1649-40f0-402b-9be8-90506bf3d6fb
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
```

cluster admin의 권한답게 모든 권한을 가지고 있고, namespace 설정이 별도로 존재하지 않는 것을 확인할 수 있다.


### 3. admin ClusterRoleBinding 조회
```
$ kubectl get clusterrolebindings.rbac.authorization.k8s.io | grep admin
cluster-admin                                          ClusterRole/cluster-admin                                                          139d
gitlab-admin                                           ClusterRole/cluster-admin                                                          136d
kubernetes-dashboard-rolebinding                       ClusterRole/cluster-admin                                                          13d
tiller-admin                                           ClusterRole/cluster-admin                                                          136d
```

### 4. cluster-admin ClusterRoleBinding 설정 조회

`cluster-admin` ClusterRoleBinding의 설정을 조회해보자.

```
$ kubectl get clusterrolebindings.rbac.authorization.k8s.io  cluster-admin -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-09-08T07:35:35Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:rbac.authorization.kubernetes.io/autoupdate: {}
        f:labels:
          .: {}
          f:kubernetes.io/bootstrapping: {}
      f:roleRef:
        f:apiGroup: {}
        f:kind: {}
        f:name: {}
      f:subjects: {}
    manager: kube-apiserver
    operation: Update
    time: "2020-09-08T07:35:35Z"
  name: cluster-admin
  resourceVersion: "102"
  uid: 63895ee2-8025-4086-b953-865899b4702e
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

`system:masters` 대상에게 권한을 부여하는 것을 볼 수 있다. `system:masters`에 대한 자세한 내용은 [쿠버네티스 공식 문서](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)를 참고한다.