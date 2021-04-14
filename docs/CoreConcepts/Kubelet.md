# Kubelet

#### Kubelet은 kubernetes 클러스터로 접촉하기 위한 유일한 중간매체
- **`kubelet`** 은 Node에 Pod를 생성하고, Scheduler는 Pod가 어디로 갈 것인지만 결정
- kubelet은 선박의 선장과 같은 역할을 한다.
- kubelet은 워커 노드에 POD를 생성하기 위해 컨테이너 런타임에 요청을 전달한다.
- kubelet은 POD와 컨테이너의 상태를 지속적으로 모니터링하고 그 결과를 kube-apiserver에 주기적으로 전송한다.
- kubeadm으로 클러스터의 워커 노드를 구성할 경우 kubelet이 자동으로 설치되고 실행되는 것은 아니다. 반드시 별도로 kubelet을 다운로드하고 설치 후 실행해야 한다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kubelet.PNG>
  
## Install kubelet
- Kubeadm은 기본적으로 kubelet을 배포하지 않습니다. 수동으로 다운로드하여 설치해야함
- Download the kubelet binary from the kubernetes release pages
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
  ```
- Extract it
- Run it as a service

  
## View kubelet options
- 실행중 Process 및 영향 옵션 보기
  ``` 
  $ ps -aux |grep kubelet
  ```

K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/08-Kubelet.md
https://github.com/jonnung/cka-practice/blob/master/1-core-concepts.md
