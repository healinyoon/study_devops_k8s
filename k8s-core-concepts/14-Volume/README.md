# Volume

### 특징
* Container가 외부 스토리지에 액세스하고 공유하는 방법
* Pod의 각 Container에는 고유의 분류된 파일 시스템이 존재
* Volume은 Pod 컴포넌트이며, Pod의 `spec`에 정의(독립적인 Kuberentes object가 아니며 스스로 생성, 삭제 불가)
* 각 Container의 파일 시스템의 Volume을 마운트하여 생성

### Volume의 종류

| 구분 | 목록 | 특징 |
| --- | --- | --- |
| 임시 Volume | emptyDir | * 임시 Volume으로 Containerr가 삭제되면 함께 삭제됨<br/>* 데이터 유지가 아닌 Container끼리 데이터를 공유하기 위한 Volume |
| 로컬(=Node) volume | hostpath<br/>local | * Pod가 떠있는 Node에서만 보관됨<br/>* 데이터 유지가 아닌 Node 관리를 목적으로 하는 Volume(Node와 데이터를 공유하기 위한 Volume) |
| 네트워크 Volume | iSCSI<br/>NFS<br/>cephFS<br/>glusterFS... | 외부와 데이터를 공유하기 위한 Volume |
| 네트워크 Volume<br/>(클라우드 종속적) | gcePersistentDisk<br/>awsEBS<br/>azureFile | |