# Volume

### 특징
* Container가 외부 스토리지에 액세스하고 공유하는 방법이다.
* Pod의 각 Container에는 고유의 분류된 파일 시스템이 존재한다.
* Volume은 Pod 컴포넌트이며, Pod의 `spec`에 정의(독립적인 Kuberentes object가 아니며 스스로 생성, 삭제 불가)한다.
* 각 Container의 파일 시스템의 Volume을 마운트하여 생성한다.

### Volume의 종류

| 구분 | 목록 | 특징 |
| --- | --- | --- |
| 임시 Volume | emptyDir | * 임시 Volume으로 Containerr가 삭제되면 함께 삭제됨<br/>* 데이터 유지가 아닌 Container끼리 데이터를 공유하기 위한 Volume |
| 로컬(=Node) Volume | hostpath<br/>local | * Pod가 떠있는 Node에서만 보관됨<br/>* 데이터 유지가 아닌 Node 관리를 목적으로 하는 Volume(Node와 데이터를 공유하기 위한 Volume) |
| 네트워크 Volume | iSCSI<br/>NFS<br/>cephFS<br/>glusterFS... | 외부와 데이터를 공유하기 위한 Volume |
| 네트워크 Volume<br/>(클라우드 종속적) | gcePersistentDisk<br/>awsEBS<br/>azureFile | |

# emptyDir Volume을 활용한 파일 시스템 공유

### 사례 1: 공유 스토리지가 없는 동일한 Pod의 3개의 Container

![](/k8s-core-concepts/images/12-Network-11.png)  
이미지 출처: 인프런 - devops를 위한 kubernetes 마스터

### 사례 2: 2개의 볼륨을 공유하는 3개의 Container

![](/k8s-core-concepts/images/12-Network-11.png)  
이미지 출처: 인프런 - devops를 위한 kubernetes 마스터

# emptyDir Volume 실습

### Volume을 공유하는 애플리케이션 생성(실습용 애플리케이션)

* 애플리케이션 코드 일부
```
for i in $SET

do
    echo "Running loop seq "$i > /var/htdocs/index.html
    sleep 10
done
```

10초에 한 번씩 데이터를 생성하여 웹서비스와 공유할 수 있는지 테스트

### emptyDir Volume 사용

* emptyDir Volume을 공유하는 2개의 Container를 띄우는 Pod yaml 생성

[kubernetes docs] > "volume" 검색 > [emptyDir] 선택
![](/k8s-core-concepts/images/12-Network-11.png) 

```
apiVersion: v1
kind: Pod
metadata:
  name: count
spec:
  containers:
  - image: gasbugs/count
    name: html-generator
    volumeMounts:
    - mountPath: /var/htdocs
      name: html
  - image: httpd
    name: web-server
    volumeMounts:
    - mountPath: /usr/local/apache2/htdocs
      name: html
      readOnly: true    <-- html-generator만 Write가 가능해짐
    ports:
    - containerPort: 80
  volumes:
  - name: html
    emptyDir: {}
```

* emptyDir Volume을 공유하는 2개의 Container를 띄우는 Pod yaml 실행

```
$ kubectl create -f count-httpd.yaml
pod/count created
```

* 확인

```
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
count                      2/2     Running   0          4m12s   10.40.0.2   worker1   <none>           <none>
```

* 다른 Pod에서 웹서버로 요청 보내서 테스트

http-go Pod 사용(없으면 하나 만드세요)
```
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
count                      2/2     Running   0          4m12s   10.40.0.2   worker1   <none>           <none>
http-go-5c6f458dc9-m97w8   1/1     Running   0          23h     10.32.0.4   worker2   <none>           <none>
```

요청시 응답 확인
```
$ kubectl exec -it http-go-5c6f458dc9-m97w8 -- curl 10.40.0.2
Running loop seq 27
```

# hostPath Volume을 활용한 파일 시스템 공유