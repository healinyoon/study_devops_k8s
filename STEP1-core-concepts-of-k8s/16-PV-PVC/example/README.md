# PV, PVC 실습

### PV, PVC 실습을 위해 생성해야하는 것 3가지

1) PV
2) PVC
3) Pod


# PV, PVC, Pod 생성을 위한 YAML 작성

### YAML 생성

```
$ touch pv-pvc-pod.yaml
```

### YAML에 PV 추가

PV의 `spec.nfs`와 스토리지 매칭

> pv-pvc-pod.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
spec:
  capacity:
    storage: 1Gi                
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce        
    - ReadOnlyMany      
  persistentVolumeReclaimPolicy: Retain  
  storageClassName: ""
  nfs:                      <-- PV의 spec.nfs <=> 스토리지
    server: 10.1.11.9
    path: /home/nfs
```

### YAML에 PVC 추가

* PVC의 `spec.accessModes`와 PV의 `spec.accessModes` 매칭
* PVC의 `spec.resources`와 PV의 `spec.capacity` 매칭

> pv-pvc-pod.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc                    
spec:
  accessModes:              <-- PVC의 spec.accessModes <=> PV의 spec.accessModes 
    - ReadWriteOnce             
  volumeMode: Filesystem
  resources:                <-- PVC의 spec.resources <=> PV의 spec.capacity
    requests:
      storage: 1Gi              
  storageClassName: ""         
```

### YAML에 jenkins Pod 추가([참고](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#%EB%B3%BC%EB%A5%A8%EC%9C%BC%EB%A1%9C-%ED%81%B4%EB%A0%88%EC%9E%84%ED%95%98%EA%B8%B0))

Pod의 `spec.volumes.name.persistentVolumeClaim.claimName`와 PVC의 `metadata.name` 매칭

> pv-pvc-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-pod
spec:
  containers:
    - name: jenkins
      image: jenkins
      volumeMounts:
      - mountPath: "/var/jenkins_home"
        name: jenkins
  volumes:
    - name: jenkins
      persistentVolumeClaim:
        claimName: test-pvc             <-- Pod의 spec.volumes.name.persistentVolumeClaim.claimName <=> PVC의 metadata.name
```

# YAML 실행 및 쿠버네티스 리소스 동작 확인

* YAML 실행
```
$ kubectl create -f pv-pvc-pod.yaml
pod/jenkins-pod created
persistentvolumeclaim/test-pvc created
persistentvolume/test-pv created
```

* PV 확인

`STATUS`가 `Bound`이므로 스토리지와 정상 연동 확인

```
$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
test-pv   1Gi        RWO            Retain           Bound    default/test-pvc                           50s
```

* PVC 확인

`STATUS`가 `Bound`이므로 PV와 정상 연동 확인

```
$ kubectl get pvc
NAME       STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test-pvc   Bound    test-pv   1Gi        RWO                           55s
```

* Pod 확인
```
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP          NODE      NOMINATED NODE   READINESS GATES
jenkins-pod                1/1     Running   0          4m45s   10.40.0.2   worker1   <none>           <none>
```

* nfs 스토리지 데이터 확인
```
$ ls /home/nfs/
config.xml                     init.groovy.d                        nodeMonitors.xml          secrets      war
copy_reference_file.log        jenkins.CLI.xml                      nodes                     test.txt
hudson.model.UpdateCenter.xml  jenkins.install.UpgradeWizard.state  plugins                   updates
identity.key.enc               jobs                                 secret.key                userContent
index.html                     logs                                 secret.key.not-so-secret  users
```
