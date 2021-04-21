# Cluster Upgrade Introduction
- 모든 쿠버네티스 구성 요소는 다른 릴리즈 버전을 가질 수 있다.
- Kube API 서버는 컨트롤 플레인의 핵심 구성요소이며, 다른 구성 요소보다 Kube API 서버 버전이 높아야 한다.
- Cotroller-manager와 Scheduler는 Kube API 버전보다 1개까지 낮아도 괜찮고, kubelet과 Kube Proxy는 2개 버전까지 낮아도 괜찮다.
- kubectl 은 Kube API 버전보다 1개 높거나 같아야 한다.
- kube-apiserver 버전이 v1.10일 때 --> Controller-manager, kube-scheduler 버전은 v1.9 또는 v1.10 가능하고, kubelet, kube-proxy는 v1.8, v1.9 또는 v1.10으로 가능하다.
  
#### 언제든지 kubernetes는 최신 3 개의 마이너 버전 만 지원합니다
- 쿠버네티스 클러스터를 업그레이드 해야 할 시점은 언제인가?
  --> 쿠버네티스 그룹은 최근 minor 버전 3개까지 공식 지원하기 때문에 현재 사용중인 쿠버네티스 버전보다 3단계 높은 릴리즈가 출시된 경우에 업그레이드를 고려할 수 있다.
- 권장되는 접근 방식은 한 번에 하나의 마이너 버전을 업그레이드하는 것
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/up2.PNG>
  
#### Options to upgrade k8s cluster
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/opt.PNG>
  
## Upgrading a Cluster
- 클러스터 업그레이드에는 두 가지 주요 단계가 포함됩니다
  
#### 워커 노드를 업그레이드하는 데 사용할 수있는 다양한 전략이 있습니다
- 하나는 한 번에 모두 업그레이드하는 것입니다. 하지만 그러면 Pod가 다운되고 사용자는 애플리케이션에 액세스 할 수 없습니다.
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/stg1.PNG>
- 두 번째는 한 번에 하나의 노드를 업그레이드하는 것입니다. 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/stg2.PNG>
- 세 번째는 클러스터에 새 노드를 추가하는 것입니다.
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/stg3.PNG>
  
## kubeadm - Upgrade master node
- kubeadm에는 클러스터 업그레이드에 도움이되는 업그레이드 Command가 있습니다.
- kubeadm의 upgrade plan 명령어를 실행하면 업그레이드 하는 데 도움이 되는 정보를 알려준다
- 클러스터 버전을 업그레이드 하기 전에 kubeadm 버전 먼저 올려준 뒤 작업해야 한다. 만약 현재 클러스터 버전이 v1.11일 때 v1.13으로 업그레이드를 하려고 한다면 kubeadm버전을 v1.12로 업그레이드해서 1단계씩 업그레이드를 진행해야 한다.
  ```
  $ kubeadm upgrade plan
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kube1.png>
  
- Upgrade kubeadm from v1.11 to v1.12
  ```
  $ apt-get upgrade -y kubeadm=1.12.0-00
  ```
- Upgrade the cluster
  ```
  $ kubeadm upgrade apply v1.12.0
  ```
- 'kubectl get nodes'명령을 실행하면 이전 버전이 표시됩니다. 이는 명령의 출력에서 API 서버 자체의 버전이 아니라 API 서버에 등록 된 각 노드의 kubelet 버전을 표시하기 때문입니다.  
  ```
  $ kubectl get nodes
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kubeu.PNG>
  
- Upgrade 'kubelet' on the master node
  ```
  $ apt-get upgrade kubelet=1.12.0-00
  ```
- Restart the kubelet
  ```
  $ systemctl restart kubelet
  ```
- Run 'kubectl get nodes' to verify
  ```
  $ kubectl get nodes
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kubeu1.PNG>
 
## kubeadm - Upgrade worker nodes
  
- 마스터 노드에서 'kubectl drain'명령을 실행하여 워크로드를 다른 노드로 이동합니다.
  ```
  $ kubectl drain node-1
  ```
- Upgrade kubeadm and kubelet packages
  ```
  $ apt-get upgrade -y kubeadm=1.12.0-00
  $ apt-get upgrade -y kubelet=1.12.0-00
  ```
- 새 kubelet 버전에 대한 노드 구성 업데이트
  ```
  $ kubeadm upgrade node config --kubelet-version v1.12.0
  ```
- Restart the kubelet service
  ```
  $ systemctl restart kubelet
  ```
- Mark the node back to schedulable
  ```
  $ kubectl uncordon node-1
  ```
  
  <img src= https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kubeu2.PNG>
  
- Upgrade all worker nodes in the same way

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kubeu3.PNG>
  

#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/
  
  
  
