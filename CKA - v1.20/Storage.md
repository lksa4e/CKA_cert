# Understand storage classes, persistent volumes
Storage Class - https://kubernetes.io/docs/concepts/storage/storage-classes/ <br>
Persistent Volume - https://kubernetes.io/docs/concepts/storage/persistent-volumes/


`StorageClass`는 관리자가 제공하는 스토리지의 "클래스"를 설명하는 방법을 제공합니다. 클래스는 서비스 품질 수준, 백업 정책 또는 클러스터 관리자가 결정한 임의의 정책에 매핑 될 수 있습니다.

### AWS ebs Storage Class의 예시:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

#### Key declarations are:

* `provisioner` : 사용할 볼륨 플러그인을 결정합니다. 일반적으로 클라우드 제공 업체 및 제공하는 특정 스토리지 서비스와 일치합니다.

* `parameters` : 기본 제공자의 맥락에서이 스토리지 클래스의 특성을 설명합니다. 이 예에서`type`은`gp2`이며, AWS 용어로는 범용 SSD를 의미합니다. 다른 유형으로는 'IO1'(프로비저닝 된 IOPS), 'ST1'(처리량 최적화) 및 'STC'(콜드 스토리지)가 있습니다.

`storageclass` 객체는 자체적으로 *호출 될 때* 프로비저닝되는 스토리지를 정의합니다. 그 자체로는 아무것도하지 않습니다.

`persistentvolume` 객체는`storageclass`에서 스토리지를 요청하는 데 사용할 수 있으며 일반적으로 Pod manifest의 일부입니다.

### Standard 라는 Storage Class를 사용하는 PV 의 예시

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: standard
```
### Local Storage Class
로컬 볼륨을 사용하는 경우 --> 현재 동적 프로비저닝을 지원하지는 않지만, 파드 스케줄링까지 볼륨 바인딩을 지연시키기 위해서는 Storage Class를 사용하는 수밖에 없다. 이것은 WaitForFirstConsumer 볼륨 바인딩 모드에 의해 지정된다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

# Understand volume mode, access modes and reclaim policies for volumes

## Volume modes:

오직 2개의 option만 존재함:

* `block` - 파일 시스템 *없이* raw block device로 Pod에 마운트됩니다. Pod/Application은 raw block device를 처리하는 방법을 이해해야합니다. 이러한 방식으로 표시하면 더 복잡하지만, 보다 나은 성능을 얻을 수 있습니다.

* `filesystem` - 디렉토리 내의 포드 파일 시스템 내부에 마운트 됨. 볼륨이 파일 시스템이 없는 블록 장치로 지원되는 경우 Kubernetes는 하나를 생성합니다. 'block' device에 비해 이 방법은 성능을 희생하면서 최고의 호환성을 제공합니다.

## Access Modes

3가지 옵션 존재:

*  `ReadWriteOnce` – 볼륨은 단일 노드에 의해 read-write 로 마운트
*  `ReadOnlyMany` – 볼륨은 여러 노드에 의해 read-only 로 마운트
*  `ReadWriteMany` – 볼륨은여러노드에 의해 read-write 로 마운트

# persistent volume claims 기본 이해

`PersistentVolume` 은 관리자가 프로비저닝한 스토리지로 생각할 수 있습니다. (사전 할당)

`PersistentVolumeClaim`은 user/workload가 요청한 스토리지로 생각할 수 있습니다. 

사용자가 PVC를 작성하여 스토리지를 요청하면 Kubernetes는 해당 PVC를 사전 할당된 PV와 일치시키려고합니다. 일치하는 항목이 발견되면 PVC가 PV에 바인딩되고 사용자는 사전 할당 된 스토리지를 사용하기 시작합니다.


# PV로 애플리케이션을 구성하는 방법

Storage Class를 사용하지 않는 경우:

PV 생성: 

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual    # Storage Class를 manual로 함
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce            # pvc의 accessmode와 일치해야함
  hostPath:
    path: "/mnt/data"          # PV로 사용할 공간
```

PVC 생성:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual     #Storage Class manual
  accessModes:
    - ReadWriteOnce            # pv의 accessMode와 일치
  resources:                   # 단순 resource 정보만
    requests:
      storage: 3Gi
```

PVC를 사용하여 Volume 구성하는 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage         # PVC로부터 오는 볼륨에 붙일 이름
      persistentVolumeClaim:      
        claimName: task-pv-claim         # PVC의 이름
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"            # 컨테이너 내에서 마운트할 곳 
          name: task-pv-storage                         # 위쪽의 volumes의 name과 일치해야함! PVC로부터 오는 볼륨을 지칭하기 때문에
```

스토리지 클래스를 활용하는 경우`persistentvolume` 객체가 필요하지 않고`persistentvolumeclaim` 만 있으면됩니다.


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: myStorageClass                  # StorageClass 이름
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage            # 일치
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage          # 일치
```

