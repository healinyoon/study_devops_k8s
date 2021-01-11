# Auto Scaling HPA

### Pod Scaling의 3가지 방법

* HPA: Pod 자체를 복제하여, Pod의 개수를 늘리는 방법
* VPA: resource를 증가시켜 Pod가 사용가능한 resource를 늘리는 방법 - cloud에서 지원
    * 수직형 pod 자동 확장
    * https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler?hl=ko
* CA: 번외로 cluster 자체를 늘리는 방법(Node 추가) - cloud에서 지원
    * 클라우드 자동 확장 처리
    * https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler

### HPA(Horizontal Pod Autoscaler)

* Kubernetes의 기본 Autoscaling 기능으로 내장되어 있다.
* CPU 사용률을 모니터링하여 Pod의 개수를 늘리거나 줄인다.


# HPA 설정 방법

### 방법 1. 명령어를 사용하여 AutoScaling 저장

```
# kubectl autoscale deployment my-app --max 6 --min 4 --cpu-percent 50
```

### 방법 2. HPA YAML을 작성하여 타겟 Pod 지정

> autoscaling.yaml
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata: 
  name: myapp-hpa
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: myapp
  targetCPUUtilizationPercentage: 30
```

# 연습: php-apache 서버 구동 및 노출

[쿠버네티스 공식 사이트](https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

### 연습용 소스코드

> dockerfile
```
FROM php:5-apache
ADD index.php /var/www/html/index.php
RUN chmod a+rx index.php
```

> index.php
```
<?php
    $x = 0.0001;
    for ($i = 0; $i <= 1000000; $i++) {
        $x += sqrt($x);
    }
    echo "OK!";
?>
```

변수 x에 0~1000000까지 제곱을 더하는 작업을 반복한다.   
→ 사용자의 요청마다 위의 반복을 수행하고 OK를 반환하는데, 수행 중 cpu 사용량이 증가하게 된다.  
→ 이때 cpu 사용량이 50%가 넘어가면 Auto Scaling 되도록 구성해보자.  

### 실행

(※ 필수 참고) HPA는 `metrics-server`가 미리 지정되어 있어야 한다(STEP3-02-metric-server에서 다룬다).

#### 1. php Deployment와 Service 생성

image를 k8s에서 제공하고 있으므로 바로 kubectl 명령어 실행

```
$   kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
deployment.apps/php-apache created
service/php-apache created
```

#### 2. Auto Scaling 생성

```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/php-apache autoscaled
```

만약 yaml로 보고 싶다면,
```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 -o yaml --dry-run=client
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: php-apache
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 50
status:
  currentReplicas: 0
  desiredReplicas: 0
```

#### 3. 생성된 HPA 확인

```
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          3m43s
```

#### 4. PHP Load 증가 / 정지에 따른 HPA 변화 관찰 

먼저 HPA의 변경사항을 모니터링하기 위해 아래 명령어를 수행한다. `-w` 옵션을 주면 변경 사항이 발생할 때마다 출력해준다.
```
$ kubectl get hpa -w
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        5          7m37s
```

다음으로 새로운 cmd 창을 띄우고 PHP Load를 증가시키는 명령어를 수행한다.
```
$ kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```

OK를 출력이 반복되는 동안, 모니터링 창에서 Auto Scaling을 관찰해보자.
HPA에 지정해준 Targets의 `--cpu-percent=50` 맞추기 위해, `REPLICAS=6`까지 Auto Scaling 되는 것을 볼 수 있다.
```
$ kubectl get hpa -w
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        5          7m37s
php-apache   Deployment/php-apache   9%/50%    1         10        5          8m8s
php-apache   Deployment/php-apache   60%/50%   1         10        5          9m9s
php-apache   Deployment/php-apache   60%/50%   1         10        6          9m24s
php-apache   Deployment/php-apache   49%/50%   1         10        6          10m
php-apache   Deployment/php-apache   51%/50%   1         10        6          11m
```

마지막으로 PHP Load 증가 명령어를 종료시키고(ctrl+c), 모니터링 창에서 Auto Scaling을 관찰해보자.
resource 추가는 즉각적으로 일어나지만, resource를 회수하는 것은 천천히 진행된다.
```
$ kubectl get hpa -w
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        5          7m37s
php-apache   Deployment/php-apache   9%/50%    1         10        5          8m8s
php-apache   Deployment/php-apache   60%/50%   1         10        5          9m9s
php-apache   Deployment/php-apache   60%/50%   1         10        6          9m24s
php-apache   Deployment/php-apache   49%/50%   1         10        6          10m
php-apache   Deployment/php-apache   51%/50%   1         10        6          11m
php-apache   Deployment/php-apache   46%/50%   1         10        6          12m
php-apache   Deployment/php-apache   6%/50%    1         10        6          13m
php-apache   Deployment/php-apache   0%/50%    1         10        6          14m
php-apache   Deployment/php-apache   0%/50%    1         10        6          18m
php-apache   Deployment/php-apache   0%/50%    1         10        1          18m
```

18m에 `REPLICAS=6` -> `REPLICAS=1`로 줄어든 것을 확인할 수 있다.
