# Kube Proxy

Kubernetes Cluster에서는 모든 포드가 다른 모든 포드에 도달할 수 있음. 이는 포드 네트워킹 클러스터를 클러스터에 배포함으로써 가능함 
- Kube-Proxy는 kubernetes 클러스터의 각 노드에서 실행되는 프로세스

- 쿠버네티스 클러스터의 모든 POD는 서로 다른 POD와 통신할 수 있다. 이것이 가능한 이유는 클러스터 내 POD 네트워크를 담당하는 기능이 배포되어 있기 때문이다. 
- POD 네트워크는 클러스터 내 모든 노드에 내부적인 가상 네트워크를 통해 모든 POD가 통신할 수 있도록 해준다. 
- a better way for the web application to access the database is using a **service**.
- 웹 애플리케이션 POD에서 DB가 실행되고 있는 POD에 접근하기 위한 가장 좋은 방법은 Service를 이용하는 것이다. 
- Service는 고유한 IP를 할당 받고, FQDN을 통해 접근할 수 있기 때문에 어떤 POD에서도 접근할 수 있다. Serivce는 전달 받은 요청을 자신이 담당하는 백엔드 POD(실제 목적 Pod)로 전달하는 역할을 한다.
- Service는 어떻게 IP를 얻는 것이며, 같은 POD 네트워크에 조인할 수 있는 것인가?  --> 이것을 가능하게 해주는 것이 Kube-Proxy
  - Service는 POD 네트워크에 조인하지 않는다. 왜냐하면 Service는 실제 존재하는 것이 아니기 때문이다. 
  - POD와 같이 컨테이너로 실행되지 않으며 어떤 네트워크 인터페이스나 요청을 기다리는 프로세스가 존재하는 것이 아니다. 
  - 이것은 가상 컴포넌트 개념이며 쿠버네티스 메모리 상에 존재하는 것일뿐이다. 
- kube-proxy는 쿠버네티스 클러스터 내 모든 노드에서 실행되는 프로세스다. 
- 이것의 역할은 새로운 Service를 만들고 Serivce를 찾는 역할을 수행한다.
- 어떤 서비스가 어떤 백엔드 POD로 전달될 수 있는지 같은 네트워크 규칙을 만든다.
- 이와 같은 네트워크 규칙을 만드는 한가지 방법으로는 IPTABLES을 사용하는 것이다. 
- 모든 노드에 Service의 IP로 들어오는 트래픽에 대해 실제 POD로 보낼 수 있는 규칙을 IPTABLES에 만들어 준다.
- kubeadm은 모든 노드에 kube-proxy를 배포하며 이는 DaemonSet으로서 실행된다. 따라서 모든 클러스터의 노드마다 오직 하나의 kube-proxy POD가 실행될 수 있다.

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kube-proxy.PNG>
  
## Install kube-proxy - Manual
- Download the kube-proxy binary from the kubernetes release pages
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
  ```
- Extract it
- Run it as a service


## View kube-proxy options - kubeadm
- kubeadm 를 사용하는 경우 kubeadm는 kube-proxy를 kube-system 네임 스페이스의 Pod로 배포합니다. 사실은 마스터 노드에 데몬 셋으로 배포되는 것임.
  ```
  $ kubectl get pods -n kube-system
  ```
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kube-proxy2.PNG>
  
K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/
- https://kubernetes.io/docs/concepts/overview/components/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/09-Kube-Proxy.md
https://github.com/jonnung/cka-practice/blob/master/1-core-concepts.md
