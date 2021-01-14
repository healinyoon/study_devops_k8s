# Node 업그레이드 절차

### Node의 유지보수

Kernel 업그레이드, libc 업그레이드, 하드웨어 복구 등과 같이 **Node를 재부팅해야 하는 경우**, Node를 갑자기 내리면 다양한 문제가 발생할 수 있다. 왜냐하면 Pod는 Node가 내려가서 5분 이상 복구되지 않으면 그때서야 다른 Node에 Pod를 복제해주기 때문이다. 따라서 Node 업그레이드 프로세스를 보다 효율적으로 제어하려면, 하나의 Node씩 아래의 절차를 수행해야 한다.

![](/STEP5-cluster-operation/images/01-node-upgrade-1.png)

* Drain Node: Node Pod들을 모두 내린다. 
    * `Cordon`: Pod 봉쇄 조치로, 새로운 Pod가 해당 Node에 배포될 수도 없다.
* Update Node: Node 업데이트한다.
* Uncordon Node: 봉쇄를 해제 한다.

### Node 업그레이드 

#### 1. 현재 Node 확인

```
$ kubectl get node
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   127d   v1.19.0
worker1   Ready    <none>   127d   v1.19.0
worker2   Ready    <none>   127d   v1.19.0
```

#### 2. Node Drain(배수)

```
$ kubectl drain worker1
node/worker1 cordoned
error: unable to drain node "worker1", aborting command...

There are pending nodes to be drained:
 worker1
cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/kube-proxy-587jv, kube-system/weave-net-p964j
cannot delete Pods with local storage (use --delete-local-data to override): default/ratings-v1-7d99676f7f-7x8nl, default/reviews-v2-6c5bf657cf-n5vwl, gitlab-managed-apps/runner-gitlab-runner-69779496d5-5xjzp, istio-system/grafana-784c89f4cf-zksvs, istio-system/jaeger-7f78b6fb65-76mk8, kube-system/metrics-server-75f98fdbd5-mfb2k
```

다음과 같은 경고들이 발생한다. 

* cannot delete Pods with local storage: Container 내부의 Local Storage에 저장된 데이터는 모두 제거된다.
* cannot delete DaemonSet-managed Pods: DaemonSet의 데이터가 모두 제거된다.

Node에 스케줄링된 Pod들을 내리는 작업이므로 발생하는 경고이다.

#### 3. 옵션을 사용하여 다시 시도

옵션을 사용하여 시도하면, Drain이 진행된다.

```
$ kubectl drain worker1 --ignore-daemonsets --delete-local-data
node/worker1 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-587jv, kube-system/weave-net-p964j
evicting pod default/reviews-v2-6c5bf657cf-n5vwl
evicting pod blue/pod-jenkins-deploy-6c8f5b65cb-gmg8k
evicting pod default/ratings-v1-7d99676f7f-7x8nl
evicting pod istio-system/jaeger-7f78b6fb65-76mk8
evicting pod gitlab-managed-apps/runner-gitlab-runner-69779496d5-5xjzp
evicting pod istio-system/grafana-784c89f4cf-zksvs
evicting pod istio-system/kiali-7476977cf9-x8zlr
evicting pod kube-system/coredns-f9fd979d6-79vc6
evicting pod kube-system/metrics-server-75f98fdbd5-mfb2k
I0114 07:19:18.353345  105000 request.go:645] Throttling request took 1.083237989s, request: GET:https://10.1.11.7:6443/api/v1/namespaces/kube-system/pods/metrics-server-75f98fdbd5-mfb2k
pod/metrics-server-75f98fdbd5-mfb2k evicted
pod/pod-jenkins-deploy-6c8f5b65cb-gmg8k evicted
pod/grafana-784c89f4cf-zksvs evicted
pod/kiali-7476977cf9-x8zlr evicted
pod/runner-gitlab-runner-69779496d5-5xjzp evicted
pod/jaeger-7f78b6fb65-76mk8 evicted
pod/coredns-f9fd979d6-79vc6 evicted
pod/reviews-v2-6c5bf657cf-n5vwl evicted
pod/ratings-v1-7d99676f7f-7x8nl evicted
node/worker1 evicted
```

#### 4. Drain 후 Node 관찰

```
$ kubectl get nodes
NAME      STATUS                     ROLES    AGE    VERSION
master    Ready                      master   127d   v1.19.0
worker1   Ready,SchedulingDisabled   <none>   127d   v1.19.0
worker2   Ready                      <none>   127d   v1.19.0
```

worker1의 STATUS가 `Ready,SchedulingDisabled`로 변경되었다. 해당 Node에는 Scheduling이 되지 않을 것이다.

#### 5. Node 업그레이드 후 Node 복구(Uncordon)

Node 업그레이드를 진행한 후, Node를 다시 `Uncordon` 해준다.

```
$ kubectl uncordon worker1
node/worker1 uncordoned

ldccai@master:~$ kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   127d   v1.19.0
worker1   Ready    <none>   127d   v1.19.0
worker2   Ready    <none>   127d   v1.19.0
```

worker1의 STATUS가 `Ready`로 다시 변경되었다. 이제 Node에 Scheduling이 가능하다.


### Cordon과 Drain의 차이

* Drain: 기존의 Schedulinge된 모든 Pod를 다른 Node로 Re-Scheduling을 시도한다. 그리고 봉쇄하여 더이상 Node에 새로운 Pod가 Scheduling되지 않게 한다.
* Cordon: 기존의 모든 Pod는 그대로 유지하되, 봉쇄만 해서 Node에 새로운 Pod가 Scheduling되지 않게 한다.

### Tip !
Uncordon을 한다고 해서, 다른 Node에 Re-Scheduling된 Pod가 원래의 Node로 되돌아오지 않는다. 왜냐면 Re-Scheduling 당시 새로운 Node에서 가능하다고 판단하여 Pod가 배포되었기 때문이다. 만약 다른 모든 Node에서 Re-Scheduling이 불가능하다고 판단되면 Pod는 `Pending` 상태로 떠돌게 된다.


# 실습

적적한 Pod를 배포한 후에 Node 업그레이드를 위한 절차를 수행하며 관찰해본다.

#### 1. Node 확인
```
$ kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   128d   v1.19.0
worker1   Ready    <none>   128d   v1.19.0
worker2   Ready    <none>   128d   v1.19.0
```

#### 2. http-go deploy 생성
```
$ kubectl create deploy http-go --image=gasbugs/http-go
deployment.apps/http-go created

$ kubectl get deploy
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
http-go          1/1     1            1           13s
```

#### 3. Replicas 수정
```
$ kubectl edit deploy http-go
deployment.apps/http-go edited

아래 내용 수정
  replicas: 10
```

```
$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE    IP           NODE      NOMINATED NODE   READINESS GATES
http-go-568f649bb-4npss           2/2     Running   0          4m1s   10.40.0.1    worker1   <none>           <none>
http-go-568f649bb-fb77f           2/2     Running   0          2m1s   10.40.0.4    worker1   <none>           <none>
http-go-568f649bb-fvksv           2/2     Running   0          2m1s   10.40.0.5    worker1   <none>           <none>
http-go-568f649bb-j2mqp           2/2     Running   0          2m1s   10.32.0.20   worker2   <none>           <none>
http-go-568f649bb-jpjck           2/2     Running   0          2m1s   10.40.0.2    worker1   <none>           <none>
http-go-568f649bb-n7w9l           2/2     Running   0          2m1s   10.38.0.7    master    <none>           <none>
http-go-568f649bb-pcvr2           2/2     Running   0          2m1s   10.40.0.6    worker1   <none>           <none>
http-go-568f649bb-phpgq           2/2     Running   0          2m1s   10.40.0.3    worker1   <none>           <none>
http-go-568f649bb-v4fzb           2/2     Running   0          2m1s   10.32.0.19   worker2   <none>           <none>
http-go-568f649bb-vj6bn           2/2     Running   0          2m1s   10.38.0.8    master    <none>           <none>
```

#### 4. worker1 Node Drain
```
$ kubectl drain worker1 --delete-local-data --ignore-daemonsets
node/worker1 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-587jv, kube-system/weave-net-p964j
evicting pod default/http-go-568f649bb-fvksv
evicting pod default/http-go-568f649bb-4npss
evicting pod default/http-go-568f649bb-fb77f
evicting pod default/http-go-568f649bb-jpjck
evicting pod default/http-go-568f649bb-pcvr2
evicting pod default/http-go-568f649bb-phpgq
pod/http-go-568f649bb-fvksv evicted
pod/http-go-568f649bb-pcvr2 evicted
pod/http-go-568f649bb-fb77f evicted
pod/http-go-568f649bb-phpgq evicted
pod/http-go-568f649bb-4npss evicted
pod/http-go-568f649bb-jpjck evicted
node/worker1 evicted
```

#### 5. Node 확인
```
$ kubectl get nodes
NAME      STATUS                     ROLES    AGE    VERSION
master    Ready                      master   128d   v1.19.0
worker1   Ready,SchedulingDisabled   <none>   128d   v1.19.0
worker2   Ready                      <none>   128d   v1.19.0
```

`SchedulingDisabled` 가 추가되었다.

#### 6. Pod 관찰
```
$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
http-go-568f649bb-8bv5v           2/2     Running   0          86s     10.32.0.22   worker2   <none>           <none>
http-go-568f649bb-c4857           2/2     Running   0          85s     10.32.0.23   worker2   <none>           <none>
http-go-568f649bb-j2mqp           2/2     Running   0          5m22s   10.32.0.20   worker2   <none>           <none>
http-go-568f649bb-jbnqs           2/2     Running   0          86s     10.32.0.21   worker2   <none>           <none>
http-go-568f649bb-n7w9l           2/2     Running   0          5m22s   10.38.0.7    master    <none>           <none>
http-go-568f649bb-v4fzb           2/2     Running   0          5m22s   10.32.0.19   worker2   <none>           <none>
http-go-568f649bb-vj6bn           2/2     Running   0          5m22s   10.38.0.8    master    <none>           <none>
http-go-568f649bb-wpj2s           2/2     Running   0          86s     10.38.0.11   master    <none>           <none>
http-go-568f649bb-wzzm4           2/2     Running   0          86s     10.38.0.9    master    <none>           <none>
http-go-568f649bb-xcdmr           2/2     Running   0          86s     10.38.0.10   master    <none>           <none>
```

worker1 Node에 scheduling 되었던 Pod들이 다른 Node에 re-scheduling 되었다.

#### 7. worker1 Node Uncordon
```
$ kubectl uncordon worker1
node/worker1 uncordoned
```

#### 8. Node 확인
```
ldccai@master:~$ kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
master    Ready    master   128d   v1.19.0
worker1   Ready    <none>   128d   v1.19.0
worker2   Ready    <none>   128d   v1.19.0
```

`SchedulingDisabled`이 제거되었다.

#### 9. Pod 관찰
```
$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE      NOMINATED NODE   READINESS GATES
http-go-568f649bb-8bv5v           2/2     Running   0          86s     10.32.0.22   worker2   <none>           <none>
http-go-568f649bb-c4857           2/2     Running   0          85s     10.32.0.23   worker2   <none>           <none>
http-go-568f649bb-j2mqp           2/2     Running   0          5m22s   10.32.0.20   worker2   <none>           <none>
http-go-568f649bb-jbnqs           2/2     Running   0          86s     10.32.0.21   worker2   <none>           <none>
http-go-568f649bb-n7w9l           2/2     Running   0          5m22s   10.38.0.7    master    <none>           <none>
http-go-568f649bb-v4fzb           2/2     Running   0          5m22s   10.32.0.19   worker2   <none>           <none>
http-go-568f649bb-vj6bn           2/2     Running   0          5m22s   10.38.0.8    master    <none>           <none>
http-go-568f649bb-wpj2s           2/2     Running   0          86s     10.38.0.11   master    <none>           <none>
http-go-568f649bb-wzzm4           2/2     Running   0          86s     10.38.0.9    master    <none>           <none>
http-go-568f649bb-xcdmr           2/2     Running   0          86s     10.38.0.10   master    <none>           <none>
```

worker1이 uncordon 되었지만, Re-Scheduling되었던 Pod가 원래의 Node로 되돌아오지는 않는다.