# Storage Class

- Persistent Volume 및 Persistent Volume Claim을 생성하는 방법에 대해 논의했으며 Pod의 볼륨을 사용하여 해당 볼륨 공간을 요청하는 방법도 확인했습니다.
- GCP, AWS, Azure와 같은 클라우드 공급자로부터 볼륨을 가져 오는 경우 먼저 Cloud Provider에서 디스크를 만들어야합니다.
- 포드 정의 파일에서 정의 할 때마다 수동으로 만들어야합니다. 이를 **정적 프로비저닝** 이라고합니다. 

#### Static Provisioning

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/class18.PNG>

#### Dynamic Provisioning

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/class19.PNG>

- **Dynamic Provisioning** 은 Storage Class가 생성되었을 때 Persistent Volume을 자동으로 생성할 수 있도록 합니다. 

```
sc-definition.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: google-storage
provisioner: kubernetes.io/gce-pd
```

#### Create a Storage Class

```
$ kubectl create -f sc-definition.yaml
storageclass.storage.k8s.io/google-storage created
```

#### List the Storage Class

```
$ kubectl get sc
NAME             PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
google-storage   kubernetes.io/gce-pd   Delete          Immediate           false                  20s
```

#### Create a Persistent Volume Claim

```
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: google-storage       
  resources:
   requests:
     storage: 500Mi
```
```
$ kubectl create -f pvc-definition.yaml

```
#### Create a Pod

```
pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: web
  volumes:
    - name: web
      persistentVolumeClaim:
        claimName: myclaim
```
```
$ kubectl create -f pod-definition.yaml
```
#### Provisioner

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/class20.PNG>

#### Kubernetes Storage Class Reference Docs

- https://kubernetes.io/docs/concepts/storage/storage-classes/
- https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes#storageclasses
- https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html
