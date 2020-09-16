# 애플리케이션 롤링 업데이트

### 기존의 업데이트 방식 vs Rolling Update 방식
| 구분 | 비교 |
| --- | --- |
| 기존 | 기존 모든 포드를 삭제 후 새로운 포드 생성 => 잠깐의 다운 타임 발생 |
| Rolling Update | * 새 버전을 실행하는 동안 로드밸런서(서비스)가 구 버전 Pod와 연결<br/>* 서비스의 레이블 셀렉터를 수정하여 간단하게 수정 가능<br/>* 단, 하위 호환성을 제공해줘야함(구 버전에서 지원하던 것은 새 버전에서도 지원해야 함)|

### Rolling Update를 구현하는 방법: Deployment 생성시 Rolling Update 전략 명시

아래의 3가지 정보를 Deployment yaml 파일에 작성하여 업데이트 전략 명시
- Label Selector: 어떤 Label을 가진 Pod를 연결할 것인지에 대한 정보
- Replica 개수: 몇 개의 replica를 유지할 것인지에 대한 정보
- Pod Template: Pod 정보

예시)
```
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx-deployment
  strategy:
    rollingUpdate:
      maxSurge: 50%     
      maxUnavailable: 50%
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: nginx-deployment
    spec:
      containers:
      - name: nginx-deployment
        image: nginx:1.18
        ports:
        - containerPort: 80
```

또한 반드시 `kubectl create -f xx.yaml` 실행시 `--record=true` 옵션을 붙여줘야 백업 가능 <-- 히스토리 정보를 남기는 옵션

### Deployment Update 전략의 종류(Strategy Type)

- RollingUpdate(기본값)
  - 오래된 Pod를 하나씩 제거하는 동시에 새로운 Pod 추가
  - 요청을 처리할 수 있는 양은 그대로 유지
  - 반드시 이전 버전과 새 버전을 동시에 처리 가능하도록 설계한 경우에만 사용해야 함
 
 - Recreate
   - 새 Pod를 만들기 전에 오래된 Pod를 모두 삭제
   - 여러 버전을 동시에 실행 불가능
   - 잠깐의 다운 타임 발생

### Rolling Update 세부 전략

: Pod를 최대/최소 몇개까지 유지할 것인지 설정

- maxSurge
  - 기본값 25% 개수로도 설정이 가능
  - 최대로 추가 배포를 허용할 개수 설정
  - replica = 4개인 경우 25%이면, maxSurge = 1개로 설정됨 => 최대 5개까지 동시 Pod 운영

- maxUnavailable
  - 기본값 25% 개수로도 설정이 가능
  - 동작하지 않는 Pod의 개수 설정
  - replica = 4개인 경우 25%이면, maxSurge = 1개로 설정 => 총 개수 4-1개는 동시 Pod 운영

### Update 명령어

`set images` 명령어를 이용한 업데이트 수행
```
형식)
$ kubectl set image deploy {Deployment 명} {Deployment 내의 container 명(container가 2개 이상일 수 있기 때문)} --record=true

예시)
$ kubectl set image deploy http-go http-go=gasbugs/http-go:v2 --record=true
```

`edit` 명령어를 사용하여 Deployment yaml 파일 수정
```
형식) 
$ kubectl edit deploy {Deployment 명} --record=true

(yaml 피일 수정)

예시)
$ kubectl edit deploy http-go --record=true
```

# 업데이트를 실패하는 경우

### 업데이트를 실패하는 케이스
- 부족한 할당량(Insufficient quota): cpu, ram 등이 부족
- 레디네스 프로브 실패(Readiness probe failures): Pod가 준비되지 않은 경우
- 이미지 가져오기 오류(Image pull errors): 해당 이미지가 존재하지 않는 경우
- 권한 부족(Insufficient permission)
- 제한 범위(Limit ranges): 공간마다 할당된 자원을 초과하는 경우
- 응용 프로그램 런타임 구성 오류(Application runtime misconfiguration)

### 업데이트를 실패하는 경우에는 기본적으로 600초 후에 업데이트를 중지

yaml 파일 설정
```
spec:
  processDeadlineSeconds: 600
```

# Rollback

- 롤백을 실행하면 이전 업데이트 상태로 돌아감
- 롤백을 하여도 히스토리의 리버전 상태는 이전 상태로 돌아가지 않음

### 롤백 명령어
```
$ kubectl rollout undo deploy {deploy name}
```

### 특정 revision으로 롤백 명령어
```
$ kubectl rollout undo deploy {deploy 명} --to-revision={revision 번호}
```






