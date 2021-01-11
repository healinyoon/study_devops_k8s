# Multiple Schduler

[쿠버네티스 Multiple Scheduler 공식 문서](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

이번 포스트에서는 Mutiple Scheduler를 구현하지 않는다.
필요하다면 위의 공식문서를 따라서 구현하면 된다.

### Multi Scheduler의 필요성

* Kubernetes 기본 scheduler가 사용자의 필요에 맞지 않으면 사요자 고유의 scheduler를 구현할 수 있다.
* 기본 scheduler와 함께 여러 개의 scheduler를 동시에 실행 가능하다.
    * 각 Pod에 사용할 scheduler를 지정하는 방식도 가능하다.


### Multi Scheduler 작성

```
apiVersion: v1
kind: ServiceAccount                            <-- step 1) 애플리케이션의 계정을 생성
metadata:
  name: my-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding                        <-- step 2) 애플리케이션 계정에 권한을 부여
metadata:
  name: my-scheduler-as-kube-scheduler
subjects:
- kind: ServiceAccount
  name: my-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:kube-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-scheduler-as-volume-scheduler
subjects:
- kind: ServiceAccount
  name: my-scheduler
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: system:volume-scheduler
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    component: scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: scheduler
      tier: control-plane
  replicas: 1
  template:
    metadata:
      labels:
        component: scheduler
        tier: control-plane
        version: second
    spec:
      serviceAccountName: my-scheduler
      containers:
      - command:
        - /usr/local/bin/kube-scheduler
        - --address=0.0.0.0
        - --leader-elect=false
        - --scheduler-name=my-scheduler
        image: gcr.io/my-gcp-project/my-kube-scheduler:1.0
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10251
          initialDelaySeconds: 15
        name: kube-second-scheduler
        readinessProbe:
          httpGet:
            path: /healthz
            port: 10251
        resources:
          requests:
            cpu: '0.1'
        securityContext:
          privileged: false
        volumeMounts: []
      hostNetwork: false
      hostPID: false
      volumes: []
```