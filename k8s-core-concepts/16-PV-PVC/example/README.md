# 실습

### 생성해야하는 것 3가지

1) PV
2) PVC
3) Pod


# `pv-pvc-pod.yaml` yaml 파일 생성

### 파일 생성

```
$ touch pv-pvc-pod.yaml
```

### `pv-pvc-pod.yaml` yaml 파일에 PV 추가

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
  nfs:
    server: 10.1.11.9
    path: /home/nfs
```

### `pv-pvc-pod.yaml` yaml 파일에 PVC 추가

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc                
spec:
  accessModes:
    - ReadWriteOnce             
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi              
  storageClassName: ""         
```

### `pv-pvc-pod.yaml` yaml 파일에 Pod 추가

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
        claimName: test-pvc
```

# `pv-pvc-pod.yaml` yaml 파일 실행

* 실행
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