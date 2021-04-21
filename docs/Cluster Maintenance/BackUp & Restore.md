# Backup and Restore Methods
- 쿠버네티스에 있는 모든 리소스 데이터를 백업하는 가장 간단한 방법은 아래 명령어를 이용해서 한꺼번에 YAML 파일로 만드는 것이다.
```
$ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

## ETCD 백업 & 복구
- 쿠버네티스 공식 사이트에는 ETCD를 백업하고 복구하는 방법이 자세하게 나와있지 않다. 따라서 아래 명령어의 암기 필요
- Backup - ETCD
  ```
  ETCDCTL_API=3 etcdctl \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/snapshot-pre-boot.db
  ```
- Restore - ETCD Snapshot
  ```
  ETCDCTL_API=3 etcdctl \
    --endpoints=https://[127.0.0.1]:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --name=master \
    --data-dir /var/lib/etcd-from-backup \
    --initial-cluster-token=etcd-cluster-1 \
    --initial-cluster=master=https://127.0.0.1:2380 \
    --initial-advertise-peer-urls=https://127.0.0.1:2380 \
    snapshot restore /tmp/snapshot-pre-boot.db
  ```
- ETCD POD 재시작 kubeadm으로 클러스터를 구성하는 경우 기본적으로 ETCD가 Static POD로 실행된다.
따라서 /etc/kubernetes/manifests/etcd.yaml 파일에 ETCD POD 명세가 존재한다.
위 복구 과정에서 추가한 옵션 중 --data-dir과 --initial-cluster-token을 etcd 컨테이너 command 옵션에 추가 또는 수정해줘야 한다.
그리고 --data-dir이 변경되었기 때문에 해당 폴더에 대한 volumes와 volumeMounts 부분도 수정해야 한다. (사실 volumeMounts의 hostPath.path만 수정하면 된다)
## Backup Candidates
 
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/bc.PNG>
 
## Resource Configuration
- Imperative way
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rci.PNG>

- Declarative Way (Preferred approach)
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
      type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  ```
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rcd.PNG>
 
- 좋은 방법은 github와 같은 소스 코드 저장소에 리소스 구성을 저장하는 것입니다.

## Backup - Resource Configs

  ```
  $ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml (only for few resource groups)
  ```

- 고려해야 할 다른 많은 자원 그룹이 있습니다. 이것을 위한  **`ARK`** 또는 Heptio의 **`Velero`** 같은 도구들이 있습니다..

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/brc.PNG>
  
## Backup - ETCD
- 리소스를 백업하는 대신 ETCD 클러스터 자체를 백업하도록 선택할 수 있습니다. 
- 쿠버네티스의 모든 리소스는 ETCD에 저장된다. 주기적으로 ETCD를 백업하는 프로세스를 둔다면 장애 상황에 더 쉽게 대응할 수 있을 것이다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/be.PNG>
  
- **`etcdctl`** 유틸리티 스냅 샷 저장 명령을 사용하여 etcd 데이터베이스의 스냅 샷을 만들 수 있습니다.
  ```
  $ ETCDCTL_API=3 etcdctl snapshot save snapshot.db
  ```
  ```
  $  ETCDCTL_API=3 etcdctl snapshot status snapshot.db
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/be1.PNG>
  
## Restore - ETCD
- 나중에 etcd의 백업에서 데이터를 복구하기 위해서, 먼저 kubet-apiserver 서비스를 중단해야 합니다.
  ```
  $ service kube-apiserver stop
  ```
- Run the etcdctl snapshot restore command
- Update the etcd service
- Reload system configs
  ```
  $ systemctl daemon-reload
  ```
- Restart etcd
  ```
  $ service etcd restart
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/er.PNG>
  
- Start the kube-apiserver
  ```
  $ service kube-apiserver start
  ```
#### 모든 etcdctl commands 를 사용하여 인증을 위한 cert,key,cacert and endpoint를 지정합니다..
```
$ ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/etcd-server.crt \
  --key=/etc/kubernetes/pki/etcd/etcd-server.key snapshot save /tmp/snapshot.db
```

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/erest.PNG>
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/06-Cluster-Maintenance/07-Backup-and-Restore-Methods.md
 https://github.com/jonnung/cka-practice/blob/master/5-cluster-maintenance.md
