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