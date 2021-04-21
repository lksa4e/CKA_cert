

# Authentication
- 쿠버네티스는 근본적으로 사용자 계정을 관리하지 않는다. File, Certificates, LDAP 같은 서드 파티 인증처럼 외부 자원을 통해 인증을 한다.
- 하지만 쿠버네티스의 ServiceAccount로 관리할 수 있는 경우도 있다.
- API 서버에 액세스 할 수있는 사람은 인증 메커니즘에 의해 정의된다.

## Accounts

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth1.PNG>
  
#### end user(배포된 APP에 Access하는 사람)에 대한 클러스터 보안은 애플리케이션 자체에서 내부적으로 관리합니다.

 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/acc1.PNG>
 
- user의 2가지 타입 (end user를 제외한)
  - 관리자 및 개발자와 같은 사람
  - 클러스터에 액세스해야하는 기타 프로세스 / 서비스 또는 애플리케이션과 같은 로봇.
  

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/acc2.PNG>
  
- 모든 사용자 액세스는 apiserver에 의해 관리되며 모든 요청은 apiserver를 통해 진행됩니다.
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/acc3.PNG>
  
## Authentication Mechanisms
- 쿠버네티스는 근본적으로 사용자 계정을 관리하지 않는다. File, Certificates, LDAP 같은 서드 파티 인증처럼 외부 자원을 통해 인증을 한다.
- 하지만 쿠버네티스의 ServiceAccount로 관리할 수 있는 경우도 있다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth2.PNG>
  
## Authentication Mechanisms - Basic (static Files)
- 사용자 ID와 비밀번호가 담긴 CSV 파일을 이용하는 방식과 사용자 ID와 Token을 이용하는 방식이 있다.
- 실제 운영 환경에서 추천하지 않는 인증 방식이다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth3.PNG>
  
## kube-apiserver configuration
- kubeadm을 통해 설정 한 경우 다음 옵션을 사용하여 kube-apiserver.yaml 매니페스트 파일을 업데이트합니다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth4.PNG>
  
## Authenticate User

- API 서버에 액세스하는 동안 기본 자격 증명을 사용하여 인증하려면 curl 명령에 사용자 이름과 비밀번호를 지정합니다.
  ```
  $ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth5.PNG>
  
- user-details.csv 파일에 사용자를 특정 그룹에 할당하기 위해 추가 열을 가질 수 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/auth6.PNG>
  
## Note
 
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/note.PNG>
 
  

  
