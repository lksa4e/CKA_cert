# Cluster Architecture, Installation and Configuration
1. Manage role based access control (RBAC)
2. Use Kubeadm to install a basic cluster
3. Manage a highly-available Kubernetes cluster
4. Provision underlying infrastructure to deploy a Kubernetes cluster
5. Perform a version upgrade on a Kubernetes cluster using Kubeadm
6. Implement etcd backup and restore
<br>
RBAC - https://kubernetes.io/docs/reference/access-authn-authz/rbac/ <br>
Use Kubeadm to install a basic cluster - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/ < br>
Manage a highly-available Kubernetes cluster - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/ <br>
Provision underlying infrastructure to deploy a Kubernetes cluster - https://kubernetes.io/docs/setup/ <br>
Perform a version upgrade on a Kubernetes cluster using Kubeadm - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/ <br>
Implement etcd backup and restore - https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/ <br>

## Helpful command

```
# Display addresses of the master and services
kubectl cluster-info

# Dump current cluster state to stdout
kubectl cluster-info dump

# Check health of cluster components
kubectl get componentstatuses

# List the nodes
kubectl get nodes

# Show metrics for a given node
kubectl top node my-node

# List all pods in all namespaces, with more details
kubectl get pods -o wide --all-namespaces

# List all services in all namespaces, with more details
kubectl get svc  -o wide --all-namespaces
```

# Manage Role Based Access Control (RBAC)

Kubernetes는 RBAC 프레임워크를 구현하여 클러스터 내의 리소스에 대한 액세스를 제어하고 전체 인증, 권한 및 승인 제어 프레임워크의 일부를 구성합니다.

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/img.png>

누가(또는 무엇이) 어떤 리소스에 액세스 할 수 있는지 확인하려면 여러 단계를 실행해야합니다.

## Step 1 - Authentication (인증)

첫 번째 단계는 사용자 또는 서비스 계정이 자신을 식별하는 방법인 인증입니다. 소스에 따라, 그에 상응하는 인증 모듈이 사용됩니다. 
모든 인증은 Kube-api Server를 통해서 인증됩니다.

Auth Mechanisms
*   Static Password File --> 사용 x
*   Static Token File --> 사용 x
*   Certificates
*   Identity Services(3rd Party tool)

모든 인증은 TLS를 통한 HTTP를 통해 처리됩니다. 

## Step 2 - Authorization

사용자 또는 서비스 계정이 인증되면 요청을 승인해야 합니다.모든 인증 요청은 일종의 작업 요청으로 이어지며, 작업은 요청이 적용되는 개체와 작업이 무엇인지 정의합니다. 예를 들어, 지정된 네임스페이스에 있는 포드를 나열합니다.

기존 정책이 사용자에게 이러한 권한을 제공하면 모든 요청이 용이 해집니다.

## Steps 3 & 4 - Admission Control

승인 컨트롤 모듈은 요청을 수정하거나 거부할 수 있는 소프트웨어 모듈입니다. 승인 컨트롤 모듈은 권한 부여 모듈(Authorization)에서 사용할 수 있는 속성 외에도 생성 또는 업데이트 중인 개체의 컨텐츠에 액세스할 수 있습니다.
생성, 삭제, 업데이트 또는 연결 중인 개체(프록시)에 대해 작동하지만 읽지는 않습니다.

## `role` and `rolebindings`

RBAC 규칙을 구현하려면 주로 Kubernetes 내에서 `Role`과 `Rolebindings` 이라는 두 가지 객체 유형이 필요합니다:

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/roleandrolebindings.png>

`Role`은 단일 네임 스페이스 내의 리소스에 대한 액세스 권한을 부여합니다.

`Rolebinding`은 `Role`의 권한을 단일 네임스페이스 내의 사용자, 그룹 또는 서비스 계정에 부여합니다.

`clusterrole` and `clusterrolebindings` 은 비슷하게 작동하지만 Non-Namespaced 리소스에 대한 액세스를 분명히 제공합니다.

`kubectl api-resources --namespaced=false` 를 사용하여 네임스페이스가 지정되지 않은 리소스 유형을 확인할 수 있습니다. 
Examples include: `node`, `persistentvolume`, `storageclass` and `users`.

`Users` 는 `serviceaccounts` 또는 `users` 입니다. 전자는 일반적으로 응용 프로그램을 인증하는 데 사용되며 후자는 인간 사용자를 위한 것입니다.

To test, the below creates `namespace`, `serviceaccount`, `role` and `rolebinding` 

```yaml
apiVersion: v1
kind: Namespace
metadata:
 name: rbac-test
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
 name: rbac-test-sa
 namespace: rbac-test
 ```
 
중요한 format.  
`apiGroup` : 이를 적용 할 API 그룹을 결정합니다.
`resources`: 이것을 적용 할 리소스 유형.
`verbs`: 이러한 개체에 대해 할 수있는 작업 (ie create, delete, watch, etc)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 name: rbac-test-role
 namespace: rbac-test
rules:
 - apiGroups: [""]
   resources: ["pods"]
   verbs: ["get", "list", "watch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: rbac-test-rolebinding
 namespace: rbac-test
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: Role
 name: rbac-test-role
subjects:
- kind: ServiceAccount
  name: rbac-test-sa
  namespace: rbac-test
```

그런 다음 kubectl command로 이를 검증 할 수 있습니다. 다음은 해당 서비스 계정이 포드를 가져올 수 있으므로 yes를 반환합니다

```shell
kubectl -n rbac-test --as=system:serviceaccount:rbac-test:rbac-test-sa auth can-i get pods
yes
```

하지만 `secrets` 의 경우에, `no`를 리턴합니다.

```shell
kubectl -n rbac-test --as=system:serviceaccount:rbac-test:rbac-test-sa auth can-i get secrets
no
```

#  Use Kubeadm to install a basic cluster

kubeadm은 Kubernetes를 기존의 여러 바닐라 노드로 부트 스트랩하는 유틸리티입니다. 실행 가능한 최소 k8s 클러스터를 인스턴스화하는 데 필요한 모든 구성 요소를 포함하여 etcd 클러스터, Kubernetes 마스터 및 작업 노드를 관리합니다.

kubeadm 사용이 끝나면 얻을 수있는 것은 완전히 작동하고 기능을 수행할 수 있는 kubernetes 클러스터입니다.

Kubeadm은 다음 기능을 수행하는 Command Line 유틸리티입니다:

* **kubeadm init** to Kubernetes 제어 플레인 노드 부트 스트랩
* **kubeadm join** to Kubernetes 작업자 노드를 부트 스트랩하고 클러스터에 결합
* **kubeadm upgrade** to Kubernetes 클러스터를 최신 버전으로 업그레이드
* **kubeadm config** kubeadm v1.7.x 이하를 사용하여 클러스터를 초기화 한 경우 kubeadm 업그레이드를 위해 클러스터를 구성하는 경우
* **kubeadm token** to kubeadm 조인을위한 토큰 관리
* **kubeadm reset** to kubeadm init 또는 kubeadm join에 의해이 호스트에 대한 변경 사항을 되돌립니다.
* **kubeadm version** to kubeadm 버전 print
* **kubeadm alpha** to 커뮤니티에서 피드백을 수집하는 데 사용할 수있는 기능 세트 미리보기

## Kubeadm - Master Node Install

다음 예에서는 3x Ubuntu Server VM이 생성되었습니다.

* k8s-cl02-ms01
* k8s-cl02-wk01
* k8s-cl02-wk02

적절한 경우 노드에 컨테이너 런타임이 설치되어 있는지 확인합니다.

마스터 노드에서 필요한 바이너리를 설치합니다.

```shell
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

마스터 노드 초기화:
```shell
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
Note, --pod-network를 전달하기위한 요구 사항은 선택한 CNI에 따라 다릅니다. Flannel의 경우 이것은 필수입니다. Kubeadm은 필수 구성 요소가 없는 경우에 알려줍니다.

완료되면 메시지가 표시됩니다:

```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.10.80:6443 --token j5nqhd.cnfmnjgc68aato60 \
    --discovery-token-ca-cert-hash sha256:cbc91031c1ffa47bbea83aa1cf65e99821a1f582c4363e1a4408715bfd66bb60 
```

참고해야 할 몇 가지 중요한 정보:

*  Kubeadm이 사용자를 위해 admin kubeconfig 파일을 생성했으며, 편의를 위해 로그온한 사용자 홈 디렉토리에 복사할 것을 권장합니다.

*  Kubeadm은 아직 Pod Networking 솔루션을 배포하지 **않았습니다**. 따라서 이것은 설치 후 활동입니다

*  Kubeadm은 작업자 노드를 추가하기 위해 토큰과 함께 join 명령을 제공했습니다. 필요한 경우 이 토큰을 다시 생성 할 수 있습니다.

### missing worker node
```shell
kubeadm reset         # root
kubeadm token create --print-join-command  #root
kubeadm join 10.170.109.165:6443 --token 50epir.tdxylmeazfv07pjm --discovery-token-ca-cert-hash  # worker
sha256:ba6156f90d7263a8c17240dbe70b624b67ad4e1388197685e42697c008c596e9
kubectl get nodes # root
```


kubectl get nodes 명령을 실행하면 마스터 노드가 준비되지 않은 것을 볼 수 있습니다

```shell
NAME            STATUS     ROLES    AGE     VERSION
k8s-cl02-ms01   NotReady   master   6m20s   v1.20.2
```
kubeadm의 출력에 따라 네트워크 솔루션, 즉 flannel을 설치하십시오.

flannel이 올바르게 작동하려면 --pod-network-cidr=10.244.0.0/16을 kubeadm init에 전달해야합니다.

또한`sysctl net.bridge.bridge-nf-call-iptables = 1`을 실행하여 /proc/sys/net/bridge/bridge-nf-call-iptables를 1로 설정합니다.

Install Flannel:

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

몇 초 후에 마스터 노드가 준비됩니다.

```shell
NAME            STATUS   ROLES    AGE   VERSION
k8s-cl02-ms01   Ready    master   10m   v1.20.2

```

## Kubeadm - Install worker nodes

워커 노드의 설치 프로세스는 마스터 노드와 유사합니다.
- 유일한 예외는 "kubeadm init" 명령은 마스터에서만 실행되므로 실행하지 않습니다. 워커 node의 경우 "kubeadm join"을 사용합니다.

As prep:

* Install a container runtime
* Install the kubeadm binaries (as above)

워커 노드를 kubeadm에 의해 생성된 클러스터에 결합하려면 마스터에서 생성된 토큰과 함께 kubeadm join 명령을 사용해야합니다. 이는 마스터 노드에서 kubeadm init를 실행 한 후에 표시됩니다. 그러나 기록되거나 만료되지 않은 경우 마스터 노드에서 쉽게 다시 생성 할 수 있습니다.:

(on the master node)

```shell
david@k8s-cl02-ms01:~$ kubeadm token create --print-join-command
kubeadm join 172.16.10.80:6443 --token ht55yv.8lq69q0189xhe2ql     --discovery-token-ca-cert-hash sha256:cbc91031c1ffa47bbea83aa1cf65e99821a1f582c4363e1a4408715bfd66bb60
```

워커 노드에서 Join 사용(as root)

```shell
root@k8s-cl02-wk01:~# kubeadm join 172.16.10.80:6443 --token ht55yv.8lq69q0189xhe2ql     --discovery-token-ca-cert-hash sha256:cbc91031c1ffa47bbea83aa1cf65e99821a1f582c4363e1a4408715bfd66bb60
```
After which confirmation will be displayed:

```shell
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
유효성을 검사하려면 마스터 노드에서 kubectl get nodes를 실행하십시오.:
```shell
NAME            STATUS   ROLES    AGE     VERSION
k8s-cl02-ms01   Ready    master   50m     v1.20.2
k8s-cl02-wk01   Ready    <none>   2m10s   v1.20.2
```

#  Manage a highly-available Kubernetes cluster

이전 섹션에서는 하나의 마스터 노드와 여러 작업자 노드가있는 K8s 클러스터를 만드는 방법을 보여주었습니다 - 이것은 Control 플레인에 대한 복원력을 제공하지 않습니다. 이를위한 여러 토폴로지가 존재합니다:

## Stacked etcd

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/stacked-etcd.png>

* 여러개의 worker nodes
* 여러개의 control plane nodes fronted by a loadbalancer
* control plane에 내장된 etcd

Notes:

etcd는 쿼럼 기반입니다. 따라서 etcd와 함께 스택 된 제어 플레인 노드를 사용하는 경우 홀수를 사용해야합니다.

-------------------------------------------------------------------------------------------------------------
<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/external-etcd.png>

Notes:

* Multiple worker nodes
* Multiple control plane nodes fronted by a loadbalancer
* k8s cluster 밖의 etcd

Notes:

이 설정의 장점은 etcd이며 제어 플레인을 서로 독립적으로 확장하고 관리 할 수 있습니다. 이는 운영상의 복잡성을 희생하면서 더 큰 유연성을 제공합니다.

# Assessing cluster health

`kubectl get componentstatus`는 1.20부터 더 이상 사용되지 않습니다. 적절한 대체 방법에는 API 서버를 직접 조사하는 것이 포함됩니다. 
예를 들어 마스터 노드에서`curl -k https://localhost:6443/livez?verbose`를 실행하면 아래와 같은 결과가 출력됩니다:

```shell
[+]ping ok
[+]log ok
[+]etcd ok
[+]poststarthook/start-kube-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
.....etc
```
API 서버의 현재 상태를 나타내는`healthz`,`livez` 및`readyz`의 세 가지 엔드 포인트가 있습니다.

#  Provision underlying infrastructure to deploy a Kubernetes cluster

위의 토폴로지 선택은 프로비저닝해야하는 기본 리소스에 영향을 미칩니다. 프로비저닝 방법은 기본 클라우드 공급자에 따라 다릅니다. 
몇 가지 일반적인 관찰은 다음과 같습니다:

* 스왑 비활성화.
* HA를위한 클라우드 기능 활용 - 즉 여러 AZ 사용.
* Windows는 작업자 노드에 사용할 수 있지만 제어 영역에는 사용할 수 없습니다.

# Perform a version upgrade on a Kubernetes cluster using Kubeadm 
```shell
# At control Plane
kubectl drain controlplane --ignore-daemonsets   # control Node Drain

apt-mark unhold kubeadm && \                                     # kubeadm 버전 설치
apt-get update && apt-get install -y kubeadm=1.19.0-00 && \
apt-mark hold kubeadm

kubeadm upgrade plan                                             # kubeadm upgrade plan

kubeadm upgrade apply v1.19.0                                    # kubeadm upgrade 적용

apt-mark unhold kubelet kubectl && \                                                # kubelet, kubectl 설치
    apt-get update && apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00 && \
    apt-mark hold kubelet kubectl
    
sudo systemctl daemon-reload                                      # kubelet, kubectl 업그레이드 적용
sudo systemctl restart kubelet

kubectl uncordon controlplane                                     # controlplan Node uncordon
kubectl drain node01 --ignore-daemonsets                          # node01 drain

# At worker Node

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.19.0-00 && \
apt-mark hold kubeadm

apt-mark unhold kubelet kubectl && \                                                # kubelet, kubectl 설치
apt-get update && apt-get install -y kubelet=1.19.0-00 kubectl=1.19.0-00 && \
apt-mark hold kubelet kubectl

systemctl daemon-reload
systemctl restart kubelet 

# Back on Master Node

kubectl uncordon node01
kubectl get pods -o wide | grep gold (make sure this is scheduled on master node)

```

#  Implement etcd backup and restore

## Backing up etcd

DB의 스냅 샷을 찍어 안전한 위치에 저장:

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/etcd-backup.db
```
Verify the backup:

```shell
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 2125d542 |   364069 |        770 |  	3.8 MB |
+----------+----------+------------+------------+
```

Perform a restore:

```shell
ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 snapshot restore snapshotdb
```

참고 : https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/01-Cluster%20Architcture%2C%20Installation%20and%20Configuration.md
