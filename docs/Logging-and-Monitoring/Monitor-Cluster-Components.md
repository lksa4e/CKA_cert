# Monitor Cluster Components
#### Kubernetes에서 리소스 소비를 어떻게 모니터링합니까? 또는 더 중요한 것은 무엇을 모니터링하고 싶습니까??
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/mon.PNG>
 
## Heapster vs Metrics Server
- Heapster는 이제 더 이상 사용되지 않으며 **`metrics server`** 로 알려진 슬림 다운 버전이 형성되었습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/hpms.PNG>
  
## Metrics Server
- 쿠버네티스 클러스터 당 1개의 metrics server를 가질 수 있음
- metrics server는 in-memory 모니터링 솔루션이라서 이전 데이터를 저장해둘 수 없음(datadog 같은 다른 솔루션 사용 필요)

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ms1.PNG>

#### 이러한 노드에서 POD에 대한 메트릭은 어떻게 생성됩니까??
- 쿠버네티스의 각 워커 노드마다 실행되는 kubelet은 kube-apiserver를 통해 명령을 전달 받고 POD를 실행하는 역할을 담당한다.
- 그리고 kubelet은 서브 컴포넌트로서 cAdvisor(Container Advisor)를 포함하고 있다.
- cAdvisor는 POD로부터 성능 지표 데이터를 받아와 Metrics Server에서 사용할 수 있는 지표로 만들고, Metrics Server는 각 노드의 kubelet API에 쿼리해서 데이터를 수집해간다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ca.PNG>
  
## Metrics Server - Getting Started

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/msg.PNG>
  
- Clone the metric server from github repo
  ```
  $ git clone https://github.com/kubernetes-incubator/metrics-server.git
  ```
- Deploy the metric server
  ```
  $ kubectl create -f metric-server/deploy/1.8+/
  ```
  
- View the cluster performance
  ```
  $ kubectl top node
  ```
- View performance metrics of pod
  ```
  $ kubectl top pod
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/view.PNG>
  
  
