# Node Selectors

#### Node Selector라는 새 속성을 spec 섹션에 추가하고 레이블을 지정합니다.
- 스케줄러는 이러한 레이블을 사용하여 포드를 배치 할 올바른 노드를 일치시키고 식별합니다.
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
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/nsel.PNG>
  
- To label nodes

  Syntax
  ```
  $ kubectl label nodes <node-name> <label-key>=<label-value>
  ```
  Example
  ```
  $ kubectl label nodes node-1 size=Large
  ```
  
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ln.PNG>
  
- To create a pod definition
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
  ```
  $ kubectl create -f pod-definition.yml
  ```
  
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/nsel.PNG>
  
## Node Selector - Limitations
- 여기서 목표를 달성하기 위해 단일 레이블과 선택기를 사용했습니다. 하지만 우리의 요구 사항이 훨씬 더 복잡하다면.
  
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/nsl.PNG>
 
- 요구사항이 복잡한 경우 **`Node Affinity and Anti Affinity`** 를 사용합니다. 
  
#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector




