# Kubernetes Security Primitives

## Secure Hosts

 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sech.PNG>
  
## Secure Kubernetes
- Kube-apiserver는 쿠버네티스의 모든 운영 요소들 중 가장 중심 역할을 하며, kubectl은 kube-api를 통해서만 클러스터에 접근할 수 있다. 즉, kube-apiserver가 가장 첫번쩨 방어선이라 할 수 있다.
- 2가지 타입의 결정이 필요하다
  - 누가 Access할 수 있는지?
  - 그들이 무엇을 할 수 있는지?
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/seck.PNG>
  
- **Authentication** (Who can access? - to API Server)
	- Files - Username and password
	- Files - Username and Tokens
	- Certificates
	- External Authentication providers - LDAP
	- Service Accounts
- **Authorization** (What can they do? - 클러스터에 access 후)
	- RBAC Authorization
	- ABAC Authorization
	- Node Authrorization
	- Webhook Mode

## Authentication
- API 서버에 액세스 할 수있는 사람은 인증 메커니즘에 의해 정의됩니다.

## Authorization
- 클러스터에 대한 액세스 권한을 얻은 후 수행 할 수있는 작업은 권한 부여 메커니즘에 의해 정의됩니다.

## TLS Certificates
- All communication with the cluster, between the various components such as the ETCD Cluster, kube-controller-manager, scheduler, api server, as well as those running on the working nodes such as the kubelet and kubeproxy is secured using TLS encryption.

 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/tls.PNG>
 
## Network Policies
What about communication between applications within the cluster?

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/np.PNG>
  
 참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/07-Security/02-Kubernetes-Security-Primitives.md
