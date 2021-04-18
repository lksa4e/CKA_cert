# Taints and Tolerations
- 포드 간 관계 및 노드에 배치되는 포드를 제한하는 방법

#### Taint 및 Tolerations는 노드에서 예약 할 수있는 포드에 대한 제한을 설정하는 데 사용됩니다. 
- 노드의 특정 taint에 내성이있는 포드 만 해당 노드에서 예약됩니다.
- Taints와 Toleration은 POD가 어떤 노드로 스케줄링 될 지 결정하기 위해 사용하는 게 아니다.
   노드가 자신에게 부여된 Taints를 Toleration하는 POD만을 허용하는 것이다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/tandt.PNG>
  
## Taints
- **`kubectl taint nodes`** 커맨드를 통해 Node에 taint 지정

  Syntax
  
  ```
  $ kubectl taint nodes <node-name> key=value:taint-effect
  ```
 
  Example
  ```
  $ kubectl taint nodes node1 app=blue:NoSchedule
  ```
  
- Taint Effect : 이 Taint를 Tolerate하지 않는 POD에는 어떤 일이 벌어지는가?
	- `NoSchedule` : POD는 이 노드에 스케줄링 되지 않을 것이다. 
	- `PreferNoSchedule` :  시스템은 이 노드에 POD를 스케줄링을 하지 않으려고 노력하지만 반드시 보장하는 것은 아니다. 
	- `NoExecute` : 이 노드에 POD를 스케줄링하지 않으며, 이미 실행중인 POD가 있는 경우 Evict 된다. 

  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/tn.PNG>
  
## Tolerations
   -  **`tolerations`** 은 Pod의 Pod 정의 파일에 추가
```
     apiVersion: v1
     kind: Pod
     metadata:
      name: myapp-pod
     spec:
      containers:
      - name: nginx-container
        image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
```
    
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/tp.PNG>
    

#### Taint 및 Tolerations는 특정 노드로 이동하도록 포드에 지시하지 않습니다. 대신 특정 허용 범위가있는 포드 만 허용하도록 노드에 지시합니다.
- To see this taint, run the below command
  ```
  $ kubectl describe node kubemaster |grep Taint
  ```
 
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/tntm.PNG>
  
     
#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/06-Taints-and-Tolerations.md
https://github.com/jonnung/cka-practice/blob/master/2-scheduling.md#kube-scheduler-%EA%B0%80-%EC%8B%A4%ED%96%89%EB%90%98%EA%B3%A0-%EC%9E%88%EC%A7%80-%EC%95%8A%EC%9D%84-%EB%95%8C-pod%EB%A5%BC-%EC%A7%81%EC%A0%91-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81-%ED%95%98%EA%B8%B0
