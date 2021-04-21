# Backup and Restore Methods
- 쿠버네티스에 있는 모든 리소스 데이터를 백업하는 가장 간단한 방법은 아래 명령어를 이용해서 한꺼번에 YAML 파일로 만드는 것이다.
```
$ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

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


 
