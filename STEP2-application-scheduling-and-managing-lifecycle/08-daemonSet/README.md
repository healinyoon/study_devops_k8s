# DaemonSet: Node당 Pod 1개씩

### 개요

ReplicaController와 ReplicaSet은 무작위 Node에 Pod를 생성하는 것과 달리 **DaemonSet은 모든 Node에 Pod를 하나씩 구성한다**. 대표적인 예시로 kube-proxy는 Kubernetes가 기본적으로 모든 Node에 구성하는 DaemonSet이다.

### 예시

아래의 예시는 fluentd가 각 Node의 DaemonSet으로 배포되어 로그를 수집할 수 있도록 하는 YAML이다.

> daemonset-fluentd-elasticsearch.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

살펴볼 점은 다음과 같다.

* DaemonSet Type

```
kind: DaemonSet
```

* Master 정보도 수집하기 위한 tolerations 설정

기본적으로 container는 master에 배포될 수 없으나, `tolerations` 설정을 통해 master에도 배포되게 한다.

```
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```

* host 관리를 목적으로 하는 hostPath volume 설정

```
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

# 연습 문제

DaemonSet으로 각 Node에 http-go 배포하기

#### 1. DaemonSet YAML 작성

> daemonset-http-go.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: http-go-ds
spec:
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: http-go
        image: gasbugs/http-go
```


#### 2. DaemonSet 생성 및 확인

```
$ kubectl create -f daemonset-http-go.yaml
daemonset.apps/http-go-ds created
```

```
$ kubectl get pod -o wide
NAME                      READY   STATUS      RESTARTS   AGE    IP          NODE      NOMINATED NODE   READINESS GATES
http-go-ds-4ljxc          1/1     Running     0          25s    10.38.0.2   master    <none>           <none>
http-go-ds-6kgwm          1/1     Running     0          25s    10.40.0.6   worker1   <none>           <none>
http-go-ds-v5r26          1/1     Running     0          25s    10.32.0.5   worker2   <none>           <none>
```