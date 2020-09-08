GKE의 프로젝트 명은 그냥 설정할 경우, 고유의 프로젝트 명을 유지하기 위해 사용자의 아이디를 앞에 붙여버린다. 따라서 처음부터 본인 계정을 프로젝트 명 앞에 붙여서 고유하게 유지하는 것을 추천한다.


$ 가 뜨면 `enter`를 눌러서 실행


# Nginx 설치

### Nginx 실행

```
$ kubectl run nginx --image=nginx
```

### Service 생성

```
$ kubectl expose deployment nginx --port=80 --type=LoadBalancer
```