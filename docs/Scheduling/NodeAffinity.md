# Node Affinity
- Node Affinity 기능의 목적은 POD를 특정 노드에 배치를 제한 하기 위함이다

#### Node Affinity의 주요 기능은 포드가 특정 노드에서 호스팅되도록하는 것입니다.
- **`Node Selectors`** 만으로는 표현의 한계가 존재함
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   nodeSelector:
    size: Large
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns-old.PNG>
  
  ### Size가 Large or Medium인 Node에 배치
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values: 
              - Large
              - Medium
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/na.PNG>
  
  ### Size가 Samll이 아닌 Node에 배치
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: NotIn
              values: 
              - Small
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/na1.PNG>
  ### Size 지정이 되어있는 Node에 배치
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: Exists
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/na2.PNG>
  

## Node Affinity Types
- Node Affinity, Pod의 라이프사이클에 따른 Scheduler의 행동에 대한 정의

- Available
  - requiredDuringSchedulingIgnoredDuringExecution
  - preferredDuringSchedulingIgnoredDuringExecution
- Planned
  - requiredDuringSchedulingRequriedDuringExecution
  - preferredDuringSchedulingRequiredDuringExecution
  
  
## Node Affinity Types States
- During Scheduling
  - 현재 Pod가 존재하지 않고 이제 생성될 때
  - Required
    - 만약 Node Affinity에 맞는 Node가 없으면 스케쥴링되지 않음
  - Preferred
    - 만약 Node Affinity에 맞는 Node가 없으면 무시하고 다른곳에 스케쥴링됨

- During Execution
  - 현재 Pod가 이미 존재할 때 Node Affinity에 영향을 줄 수 있는 변화가 생겼을 때(ex. Node의 Label 변화)
  - Ignored
    - 변화가 생겨도 실행중인 Pod에는 영향을 주지 않음
  - Required (** 미래에 적용 예정)
    - 변화가 생겼을 때 Node Affinity에 맞지 않으면 실행중인 Pod를 Evict함

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/nats.PNG>
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/nats1.PNG>
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/09-Node-Affinity.md
