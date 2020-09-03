# Label 확인

```
$ kubectl get pod --show-labels
NAME      READY   STATUS    RESTARTS   AGE   LABELS
http-go   1/1     Running   0          11h   creation_method=manual,env=prod
```

# 원하는 col만 확인하기(-L 옵션)

```
$ kubectl get pod -L env
NAME         READY   STATUS    RESTARTS   AGE   ENV
http-go      1/1     Running   0          11h   prod
http-go-v2   1/1     Running   0          62s   
```

```
$ kubectl get pod -L creation_method
NAME         READY   STATUS    RESTARTS   AGE   CREATION_METHOD
http-go      1/1     Running   0          11h   manual
http-go-v2   1/1     Running   0          24m   manual-v2
```

# Label 추가하기

```
$ kubectl label pod http-go test=foo
pod/http-go labeled

$ kubectl get pod --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
http-go      1/1     Running   0          11h   creation_method=manual,env=prod,test=foo
http-go-v2   1/1     Running   0          25m   creation_method=manual-v2
```

이미 있는 key에 그냥 추가하면 다음과 같은 에러 발생
```
$ kubectl label pod http-go test=foo
pod/http-go labeled
```

그럴 경우에는 `--overwrite` 옵션을 줘야 함
```
$ kubectl label pod http-go test=foo1 --overwrite
pod/http-go labeled
```

# Label 삭제하기(- 옵션)

```
$ kubectl label pod http-go test-
pod/http-go labeled
```

확인
```
$ kubectl get pod --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
http-go      1/1     Running   0          11h   creation_method=manual,env=prod,htest=foo1  # test label이 없어짐
http-go-v2   1/1     Running   0          28m   creation_method=manual-v2
```

# Label 필터링(-l 옵션)

env label이 있는 pod만 필터링
```
$ kubectl get pod -l env
NAME      READY   STATUS    RESTARTS   AGE
http-go   1/1     Running   0          11h
```

env label이 없는 pod만 필터링
```
$ kubectl get pod -l '!env'
NAME         READY   STATUS    RESTARTS   AGE
http-go-v2   1/1     Running   0          33m
```

조건 주기
```
$ kubectl get pod -l 'env=prod'
NAME      READY   STATUS    RESTARTS   AGE
http-go   1/1     Running   0          11h
```

여러 개의 조건 주기(, 쉼표 사용)
```
$ kubectl get pod -l 'env=prod,creation_method'
NAME      READY   STATUS    RESTARTS   AGE
http-go   1/1     Running   0          11h
```

# 연습문제(nginx-pod.yaml)
```
$ kubectl create -f nginx-pod.yaml 
```

app=nginx인 label 조회
```
$ kubectl get pod -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          14m
```

app 명의 label을 기준으로 조회
```
$ kubectl get pod  -L appNAME         READY   STATUS    RESTARTS   AGE   APP
http-go      1/1     Running   0          12h   
http-go-v2   1/1     Running   0          57m   
nginx        1/1     Running   0          15m   nginx
```

label 추가 및 확인
```
$ kubectl label pod nginx team=dev1
pod/nginx labeled

$ kubectl get pod --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
http-go      1/1     Running   0          12h   creation_method=manual,env=prod,htest=foo1
http-go-v2   1/1     Running   0          59m   creation_method=manual-v2
nginx        1/1     Running   0          17m   app=nginx,team=dev1
```