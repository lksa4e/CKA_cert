# Certificate API
새로운 클러스터 관리자가 생겼을 때 쿠버네티스 마스터 노드에 저장된 Root Certificate를 이용해서 CSR을 만든 후 새로운 관리자를 위한 admin용 Client Certificate를 만들 수 있다.   
하지만 이 작업은 마스터 노드에 직접 들어와서 해야 하는 부담과 함께 관리자가 점점 늘어나거나 만료 기간이 지났을 때 마다 해야 하는 번거로움이 있을 수 있다.   
쿠버네티스는 하위 Certificate를 요청하는 작업을 할 수 있는 API를 제공한다. 이 API를 통해서 `CertificateSigningRequest`를 보내고, 관리자가 `kubectl`을 이용해 확인한 후 승인해서 전달할 수 있다.

이 과정은 크게 4가지 단계로 이뤄진다.  

1. Create CertificatesSigningRequest Object
2. Review Requests
3. Approve Requests
4. Share Certs to users

이전 과정에서 CSR을 만드는 것과 동일하게 새로운 사용자는 `openssl genrsa` 명령어로 Private Key를 만들고 CSR까지 만든다. 그 다음 쿠버네티스의 `CertificateSigningRequest`는 오브젝트를 정의한 yaml 형식의 매니페스트 파일을 작성한다.   
`CertificateSigningRequest` 오브젝트의 `request`항목에 CSR 텍스트를 입력하며, 평문 그대로 입력하지 않고 BASE64 인코딩한 결과를 입력하도록 한다.   

## CA (Certificate Authority)
- CA는 실제로 우리가 생성 한 키와 인증서 파일의 쌍일 뿐이며, 이러한 파일 쌍에 액세스 할 수있는 사람은 누구나 kubernetes 환경에 대한 모든 인증서에 서명 할 수 있습니다.

#### Kubernetes에는 이를 수행 할 수있는 기본 제공 인증서 API가 있습니다. 
- 인증서 API를 사용하여 이제 API 호출을 통해 인증서 서명 요청 (CSR)을 kubernetes에 직접 보냅니다.
   
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csr.PNG>
   
#### 이 인증서를 추출하여 사용자와 공유 할 수 있습니다.
- A user first creates a key
  ```
  $ openssl genrsa -out jane.key 2048
  ```
- Generates a CSR
  ```
  $ openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
  ```
- 관리자에게 요청을 보내고 관리자는 키를 가져 와서 종류가 "CertificateSigningRequest"이고 인코딩 된 "jane.csr"인 CSR 개체를 만듭니다.
  ```
  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: jane
  spec:
    groups:
    - system:authenticated
    usages:
    - digital signature
    - key encipherment
    - server auth
    request:
      <certificate-goes-here>
  ```
  ```
  $ cat jane.csr |base64 
  $ kubectl create -f jane.yaml
  ```
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csr1.PNG>
  
- To list the csr's
  ```
  $ kubectl get csr
  ```
- Approve the request
  ```
  $ kubectl certificate approve jane
  ```
  
생성된 Certificate는 yaml 형식으로 가져온 다음 `status.certificate` 속성값을 다시 BASE64으로 디코딩한 결과를 사용하면 된다. 

- To view the certificate
  ```
  $ kubectl get csr jane -o yaml
  ```
- To decode it
  ```
  $ echo "<certificate>" |base64 --decode
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csr2.PNG>
  
#### 모든 인증서 관련 작업은 controller manager가 수행합니다. 
- 누구든지 인증서에 서명해야하는 경우 CA 서버, 경로 인증서 및 개인 키가 필요합니다. 컨트롤러 관리자 구성에는이를 지정할 수있는 두 가지 옵션이 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csr3.PNG>
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csr4.PNG>
  
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
 
 참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/07-Security/11-Certificate-API.md
 https://github.com/jonnung/cka-practice/blob/master/6-security.md
  

