# 초기 명령어 및 arguments 전달과 실행

Pod를 생성할 때 `spec.containers.command`와 `spec.containers.args`에 실행하기 원하는 인자를 전달하면 container가 부팅된 뒤 실행된다.

### 예시

> pod-command-args.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
restartPolicy: OnFailure
```

### 환경 변수를 활용하여 출력할 때는 $을 사용하여 명령 내용 변경 가능

```
env:
- name: MESSAGE
  value: "hello world"
command: ["/bin/echo"]
args: ["$(MESSAGE)"]
```

# 연습 문제

* busybox 이미지를 사용하는 busybox Pod를 만들어라
* busybox Pod가 유지 되는가? 그렇지 않다면 그 이유는 무어인가?
* busybox를 장시간 유지하기 위해 장시간 sleep하는 command와 args를 추가하여 실행하라.
* busybox가 계속 유지 될 수 있는가? shell에 접속하여 확인해보라.