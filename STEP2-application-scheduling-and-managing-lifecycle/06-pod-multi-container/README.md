# 한 Pod에 멀티 Container

하나의 Pod에 다수의 Container를 사용하는 방법으로 다음과 같은 특징이 있다. 
* 하나의 Pod를 사용하는 경우 같은 네트워크 인터페이스와 IPC, 볼륨 등을 공유한다.
* Pod는 효율적으로 통신하여 데이터의 지역성을 보장하고, 여러 개의 응용 프로그램이 결합된 형태로 하나의 Pod를 구성할 수 있다.
* 리소스 모니터링 용도로 함께 구성하는 경우가 많다.
    * 예시
![](/STEP2-application-scheduling-and-managing-lifecycle/images/06-pod-multi-container-1.png)

> pod-multi-container.yaml
```
apiVersion: v1
kind: Pod
metadata: 
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-date/index.html"]
```

# 연습 문제
하나의 Pod에서 nginx와 redis 이미지를 모두 실행하는 YAML을 만들고 실행하라.

### 문제 풀이

#### 1. Pod YAML 작성

> pod-nginx-redis.yaml
```
apiVersion: v1
kind: Pod
metadata: 
  name: nginx-redis
spec:
  restartPolicy: Never
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  - name: redis-container
    image: redis
```

#### 2. Pod 생성
```
$ kubectl create -f pod-nginx-redis.yaml
pod/nginx-redis created
```

#### 3. Pod 생성 확인
다음과 같이 2개의 컨테이너(2/2)가 모두 생성되는 것을 확인할 수 있다.
```
# kubectl get pod -w
NAME                      READY   STATUS              RESTARTS   AGE
nginx-redis               0/2     ContainerCreating   0          10s
nginx-redis               2/2     Running             0          26s
```

### 참고: docker container 확인

#### 1. Pod가 올라와있는 worker node 알아내기
```
$ kubectl get pod -o wide
NAME                      READY   STATUS      RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
nginx-redis               2/2     Running     0          8m18s   10.32.0.6   worker2   <none>           <none>
```

#### 2. worker2에 접속하여 docker container 확인

> 팁! 명령어 안잘리고 조회하는 법 `--no-trunc` 옵션 사용

```
# docker ps --no-trunc

CONTAINER ID                                                       IMAGE                                                                                     COMMAND                                                                                                                                                                                                                                                                                                            CREATED             STATUS              PORTS               NAMES
cd99f4b405cb8c269657ecbe1b674112ff3ce49758b8b8fcf88b5adb56c4305f   redis@sha256:0f724af268d0d3f5fb1d6b33fc22127ba5cbca2d58523b286ed3122db0dc5381             "docker-entrypoint.sh redis-server"                                                                                                                                                                                                                                                                                11 minutes ago      Up 11 minutes                           k8s_redis-container_nginx-redis_default_63467057-59fc-4ea0-bcf9-9a0e74a5bb62_0
f1ef31e22990b022819382e5006bd4bf8e92d23213a94e846429fc16fc1090e8   nginx@sha256:4cf620a5c81390ee209398ecc18e5fb9dd0f5155cd82adcbae532fec94006fb9             "/docker-entrypoint.sh nginx -g 'daemon off;'"                                                                                                                                                                                                                                                                     11 minutes ago      Up 11 minutes                           k8s_nginx-container_nginx-redis_default_63467057-59fc-4ea0-bcf9-9a0e74a5bb62_0
1313173946fd6d578e972691fddc4d2e9d4adb05b372c9e27c30bd93c6617c14   k8s.gcr.io/pause:3.2                                                                      "/pause"                                                                                                                                                                                                                                                                                                           11 minutes ago      Up 11 minutes                           k8s_POD_nginx-redis_default_63467057-59fc-4ea0-bcf9-9a0e74a5bb62_0

```