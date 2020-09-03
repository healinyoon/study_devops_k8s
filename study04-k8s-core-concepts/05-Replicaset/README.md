# 레플리케이션 컨트롤러

- 구 버전(v1.8 이전)의 쿠버네티스에서 사용
- pod가 항상 실행되도록 유지하는 쿠버네티스 리소스
  - 노드가 클러스터에서 사라지는 경우 해당 pod를 감지하고 대체 pod 생성
  - 실행 중인 pod 목록 지속적 모니터링, 실제 pod 수가 원하는 수와 항상 일치하는 지 확인

### 레플리케이션 컨트롤러의 세 가지 요소 

1) 레이블 셀렉터: 레플리케이션컨트롤러가 관리하는 포드 범위를 결정(rc는 label로 pod를 관리하기 때문)
2) 복제본 수: 실행해야하는 포드 수 결정
3) 포드 템플릿: 새로운 포드의 모양을 설명

### 레플리케이션 컨트롤러의 장점

- 포드가 없는 경우 새 포드를 항상 실행
- 노드에 장애 발생 시 다른 노드에 복제본 생성
- 수동, 자동으로 수평 스케일링

# 레플리케이션 컨트롤러 실습

레플리케이션 생성
```
$ kubectl create -f http-go-rc.yaml 
replicationcontroller/http-go created
```

레플리케이션 확인
```
$ kubectl get rc
NAME      DESIRED   CURRENT   READY   AGE
http-go   3         3         0       6s
```

pod 확인
```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-2dqcs   1/1     Running   0          62s
http-go-jl7gp   1/1     Running   0          62s
http-go-nk5jl   1/1     Running   0          62s
```

임의로 pod 제거
```
$ kubectl delete pod http-go-2dqcs
pod "http-go-2dqcs" deleted
```

자동으로 복구 되는 것 확인
```
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
http-go-76jhc   1/1     Running   0          2m3s
http-go-jl7gp   1/1     Running   0          4m55s
http-go-nk5jl   1/1     Running   0          4m55s
```

Label 확인
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          3m9s    app=http-go
http-go-jl7gp   1/1     Running   0          6m1s    app=http-go
http-go-nk5jl   1/1     Running   0          6m1s    app=http-go
http-go-v2      1/1     Running   0          3h43m   creation_method=manual-v2
```

Label  제거
```
$ kubectl label pod http-go-76jhc app-
pod/http-go-76jhc labeled 
```

Label이 제거 되어 관리되지 않게 됨 => 제거되었다고 생각하고 새로 복구 해줌
```
$ kubectl get pod --show-labels
NAME            READY   STATUS    RESTARTS   AGE     LABELS
http-go-76jhc   1/1     Running   0          4m31s   <none>
http-go-gdl6h   1/1     Running   0          30s     app=http-go
http-go-jl7gp   1/1     Running   0          7m23s   app=http-go
http-go-nk5jl   1/1     Running   0          7m23s   app=http-go
http-go-v2      1/1     Running   0          3h44m   creation_method=manual-v2
```

rc가 어느 노드에 있는지 확인
```
$ kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP          NODE           NOMINATED NODE   READINESS GATES
http-go-gdl6h   1/1     Running   0          5m21s   10.32.0.5   k8s-worker02   <none>           <none>
http-go-jl7gp   1/1     Running   0          12m     10.40.0.3   k8s-worker01   <none>           <none>
http-go-nk5jl   1/1     Running   0          12m     10.40.0.1   k8s-worker01   <none>           <none>
```

노드가 강제로 연결이 끊길 경우 관찰
```
$ kubectl get nodes -w  # 이렇게 해놓고, 네트워크를 끊어보고 관찰
```
