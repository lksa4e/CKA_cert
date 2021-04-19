# Monitor Cluster Components
#### Kubernetes에서 리소스 소비를 어떻게 모니터링합니까? 또는 더 중요한 것은 무엇을 모니터링하고 싶습니까??
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/mon.PNG>
 
## Heapster vs Metrics Server
- Heapster는 이제 더 이상 사용되지 않으며 **`metrics server`** 로 알려진 슬림 다운 버전이 형성되었습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/hpms.PNG>
  
## Metrics Server

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ms1.PNG>

#### 이러한 노드에서 POD에 대한 메트릭은 어떻게 생성됩니까??

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
  
  
