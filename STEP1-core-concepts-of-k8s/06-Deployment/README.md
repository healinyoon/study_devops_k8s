# Deployment

- Replicaset과 Replication controller 상위에서 배포되는 리소스
- 다수의 Replicaset을 다룰 수 있음
- 애플리케이션을 다운 타임 없이 업데이트 가능하도록 도와주는 리소스

### 모든 Pod를 업데이트하는 방법
방법 1) recreate: 새로운 포드를 실행시키고, 작업이 완료되면 오래된 포드를 삭제 => 잠깐의 다운타임 발생
방법 2) RollinUpdate(참고: 07-RollingUpdate에서 다룸)


### Deployment yaml 파일 작성 및 실행

예시)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  labels:
    app: jenkins
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins
        ports:
        - containerPort: 8080
```

yaml 파일 실행
```
$ kubectl create -f {Deployment yaml} 

예시)
$ kubectl create -f jenkins-deployment.yaml 
deployment.apps/jenkins-deployment created
```

### 배포된 모든 애플리케이션 확인

```
$ kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/jenkins-deployment-5d7c95487d-4f49d   1/1     Running   0          75s
pod/jenkins-deployment-5d7c95487d-7qxs8   1/1     Running   0          75s
pod/jenkins-deployment-5d7c95487d-t8dxv   1/1     Running   0          75s
 
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   26h

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins-deployment   3/3     3            3           75s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-deployment-5d7c95487d   3         3         3       75s
```

# Deployment를 이용한 자동 복구 확인

### Pod 삭제 후 자동으로 복구 되는 것 확인

Pod 삭제
```
$ kubectl delete pod jenkins-deployment-5d7c95487d-4f49d
pod "jenkins-deployment-5d7c95487d-4f49d" deleted
```

Replica 설정에 따른 자동 복구 완료 확인
```
$ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
jenkins-deployment-5d7c95487d-7qxs8   1/1     Running   0          2m49s
jenkins-deployment-5d7c95487d-gv7wl   1/1     Running   0          20s
jenkins-deployment-5d7c95487d-t8dxv   1/1     Running   0          2m49s
```

### Replicaset 상세 정보 확인

```
$ kubectl describe rs {Replicaset 명}

예시)
$ kubectl describe rs jenkins-deployment-5d7c95487d
Name:           jenkins-deployment-5d7c95487d
Namespace:      default
Selector:       app=jenkins-test,pod-template-hash=5d7c95487d
Labels:         app=jenkins-test
                pod-template-hash=5d7c95487d
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 4
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/jenkins-deployment
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=jenkins-test
           pod-template-hash=5d7c95487d
  Containers:
   jenkins:
    Image:        jenkins
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  4m50s  replicaset-controller  Created pod: jenkins-deployment-5d7c95487d-t8dxv
  Normal  SuccessfulCreate  4m50s  replicaset-controller  Created pod: jenkins-deployment-5d7c95487d-4f49d
  Normal  SuccessfulCreate  4m50s  replicaset-controller  Created pod: jenkins-deployment-5d7c95487d-7qxs8
  Normal  SuccessfulCreate  2m21s  replicaset-controller  Created pod: jenkins-deployment-5d7c95487d-gv7wl
```

### Label 제거 후 새로운 Pod 생성 확인

Replicaset에 연결된 Pod의 Label 제거  -> Replicaset은 관리해야하는 replica 숫자보다 Pod가 적은 것으로 인식하게 됨
``` 
$ kubectl label pod {Pod 명} app-

예시)
$ kubectl label pod jenkins-deployment-5d7c95487d-7qxs8 app-
pod/jenkins-deployment-5d7c95487d-7qxs8 labeled
```

새로운 Pod가 자동으로 생성되는 것 확인
```
$ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
jenkins-deployment-5d7c95487d-7qxs8   1/1     Running   0          7m23s
jenkins-deployment-5d7c95487d-gv7wl   1/1     Running   0          4m54s
jenkins-deployment-5d7c95487d-lmgrw   1/1     Running   0          47s
jenkins-deployment-5d7c95487d-t8dxv   1/1     Running   0          7m23s
```

# Deployment를 이용한 Scale In/Out 

### scale 명령 사용

`scale` 명령어를 사용하여 Replicaset 개수 변경
```
$ kubectl scale deploy {Deployment 명} --replicas=5

예시)
$ kubectl scale deploy jenkins-deployment --replicas=5
deployment.apps/jenkins-deployment scaled
```

변경된 Pod 개수 확인
```
$ kubectl get pod -l app
NAME                                  READY   STATUS    RESTARTS   AGE
jenkins-deployment-5d7c95487d-dp4bx   1/1     Running   0          42s
jenkins-deployment-5d7c95487d-gv7wl   1/1     Running   0          8m4s
jenkins-deployment-5d7c95487d-lmgrw   1/1     Running   0          3m57s
jenkins-deployment-5d7c95487d-ltw2d   1/1     Running   0          42s
jenkins-deployment-5d7c95487d-t8dxv   1/1     Running   0          10m
```

### edit 명령 사용

`edit` 명령어를 사용하여 Replicaset 개수 변경
```
$ kubectl edit deploy jenkins-deployment
deployment.apps/jenkins-deployment edited
```

변경된 Pod 개수 확인
```
$ kubectl get pod -l app
NAME                                  READY   STATUS    RESTARTS   AGE
jenkins-deployment-5d7c95487d-2nsds   1/1     Running   0          25s
jenkins-deployment-5d7c95487d-dp4bx   1/1     Running   0          2m31s
jenkins-deployment-5d7c95487d-f2szt   1/1     Running   0          25s
jenkins-deployment-5d7c95487d-gv7wl   1/1     Running   0          9m53s
jenkins-deployment-5d7c95487d-lmgrw   1/1     Running   0          5m46s
jenkins-deployment-5d7c95487d-ltw2d   1/1     Running   0          2m31s
jenkins-deployment-5d7c95487d-nsllt   1/1     Running   0          25s
jenkins-deployment-5d7c95487d-q2758   1/1     Running   0          25s
jenkins-deployment-5d7c95487d-sjqpw   1/1     Running   0          25s
jenkins-deployment-5d7c95487d-t8dxv   1/1     Running   0          12m
nginx                                 1/1     Running   0          14h
```