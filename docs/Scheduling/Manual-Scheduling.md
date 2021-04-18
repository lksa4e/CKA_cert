
## How Scheduling Works
- 클러스터에 스케줄러가 없는 경우 어떻게 합니까?
  - 모든 POD에는 기본적으로 설정되지 않은 NodeName이라는 필드가 있습니다. 일반적으로 매니페스트 파일을 만들 때이 필드를 지정하지 않으며 kubernetes에서 자동으로 추가합니다.
  - NodeName이 지정되지 않은 Pod는 스케쥴러가 감지하여 자동으로 Scheduling 함
  - 쿠버네티스는 POD의 NodeName 속성을 직접 변경하는 것을 허용하지 않으며, Binding 오브젝트를 API 호출을 통해 생성해서 스케줄링을 모방할 수 있다.
    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: nginx
     labels:
      name: nginx
    spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 8080
     nodeName: node02
    ```
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sc1.png>
    
## No Scheduler
  - 노드 자체에 Pod를 수동으로 할당할 수 있습니다. 스케줄러가 없을 때 포드를 예약하려면 포드를 생성하는 동안 포드 정의 파일에 **`nodeName`** 속성을 설정해야 합니다.
    
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sc2.PNG>
    
  - Another way
    ```
    apiVersion: v1
    kind: Binding
    metadata:
      name: nginx
    target:
      apiVersion: v1
      kind: Node
      name: node02
    ```
    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: nginx
     labels:
      name: nginx
    spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 8080
    ```
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sc3.PNG>
    
    
K8s Reference Docs:
- https://kubernetes.io/docs/reference/using-api/api-concepts/
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/02-Manual-Scheduling.md
    
    
   
