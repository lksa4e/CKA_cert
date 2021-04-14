# ETCD in Kubernetes

## ETCD Datastore
- ETCD 데이터 저장소는 **`Nodes`**, **`PODS`**, **`Configs`**, **`Secrets`**, **`Accounts`**, **`Roles`**, **`Bindings`** 등과 같은 클러스터 관련 정보를 저장함
- **`kubectl get`** 명령을 실행할 때 표시되는 모든 정보는 **`ETCD Server`**에서 가져온 것임

## Setup - Manual
- 클러스터를 처음부터 설정 한 경우 ETCD  **`ETCD`** 바이너리를 직접 다운로드하여 ETCD를 배포함
- 바이너리를 설치하고 ETCD를 마스터 노드에서 서비스로 직접 구성함
  ```
  $ wget -q --https-only "https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz"
  ```

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/etcd.PNG>
  
## Setup - Kubeadm
-  **`kubeadm`**을 사용하여 클러스터를 셋업하면 kubeadm이 **`kube-system`** 네임스페이스의 POD로 etcd 서버를 배포함
  ```
  $ kubectl get pods -n kube-system
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/etcd1.PNG>

## Explore ETCD
- k8s에 의해 저장된 모든 키를 나열하는 command
  ```
  $ kubectl exec etcd-master -n kube-system etcdctl get / --prefix -key
  ```
- k8s는 특정 디렉토리 구조에 데이터를 저장하며, root 디렉토리는 **`registry`**이고, 그 아래에 **`minions`**, **`nodes`**, **`pods`**, **`replicasets`**, **`deployments`**, **`roles`**, **`secrets`** 및 기타와 같은 다양한 k8s 구성이 있음.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/etcdctl1.PNG>

## ETCD in HA Environment
   - 고가용성 환경에서는 클러스터에 여러 개의 마스터 노드가 있으며, 여러 개의 ETCD 인스턴스가 마스터 노드들에 분산되어 있음
   -  **`etcd.service`** 구성에서 올바른 매개 변수를 지정하여 etcd 인스턴스가 서로를 알고 있는지 확인해야 함. **`--initial-cluster`** 은 etcd 서비스의 여러 인스턴스를 지정해야 함
     <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/etcd-ha.PNG>

K8s Reference Docs:
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#stacked-control-plane-and-etcd-nodes
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#external-etcd-nodes

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/04-ETCD-in-Kubernetes.md
