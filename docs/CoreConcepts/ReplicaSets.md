# ReplicaSets

## What is a Replica and Why do we need a replication controller?
- 항상 일정한 개수의 Pod가 실행될 수 있도록 유지
- 로드밸런싱을 위해(여러 노드에 걸쳐서 Pod를 생성할 수 있음)

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rc.PNG>
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rc1.PNG>
  
## ReplicaSet and Replication Controller 차이점
- **`Replication Controller`** 은 **`ReplicaSet`** 으로 대체된 오래된 기술.
- **`ReplicaSet`** 는 복제를 set up 하기위한 새로운 방법.
- Replication Controller와 큰 차이점은 **`selector`** 를 지정해서 일치하는 **`labels`** 을 가진 POD에 대한 리플리카를 관리할 수 있는 점이다.

### 레플리케이션 컨트롤러(Replication Controller)
- 레플리케이션 컨트롤러는 쿠버네티스 프로젝트 초기부터 있었던 컨트롤러입니다. 앞서 말한 것처럼 레플리케이션 컨트롤러는 파드를 관리하며 파드의 개수가 항상 일정하도록 유지합니다.
- 예를 들어! 파드의 장애 혹은 다른 이유에 의해 컨테이너가 중지된 경우, 레플리케이션 컨트롤러는 이를 감지하고 새로운 파드를 실행합니다.
- 컨트롤러의 사용이 아닌 파드만 단독으로 운영하는 경우 이런 문제 상황에 대처하기 어렵습니다. 

### 레플리카 세트(Replica Set)
- 레플리케이션 컨트롤러에서 한층 발전된 컨트롤러가 레플리카 세트입니다.
- 레플리케이션 컨트롤러와 레플리카 세트는 거의 동일한 역할을 하며, 레플리카 세트는 집합 기반의 셀렉터를 지원합니다.
- 또한, 레플리카셋은 rolling-update를 지원하지 않기에 rolling-update를 위해서는 deployment를 함께 사용해야 합니다.

```
*셀렉터(selector)란?
- 레플리카 세트는 집합 셀렉터(set-based selector)를 통해 레이블을 선택할 수 있습니다.
- 집합 기반의 셀렉터는 in notin, exists 같은 연산자를 사용합니다.
```

## Creating a Replication Controller

## Replication Controller Definition File
- spec: 에는 pod definition file에서 apiVersion, kind를 제외한 나머지 부분을 template: 아래에(children) 붙여넣는다
- 이후에 template과 동일 position에(sibling) replicas 지정을 해줌


   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rc2.PNG>
  
```
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: myapp-rc
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
```
  - replication controller 생성
    ```
    $ kubectl create -f rc-definition.yaml
    ```
  - replication controllers 리스트 업
    ```
    $ kubectl get replicationcontroller
    ```
  - replication controller에 의해 실행된 pod는 replication controller의 이름이 붙어있음
    ```
    $ kubectl get pods
    ```
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rc3.PNG>
    
## Creating a ReplicaSet
  
## ReplicaSet Definition File
- apiVersion은 RC와 다르게 apps/v1임
- spec 부분은 rc와 동일하지만, selector가 존재함
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rs.PNG>

```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
     selector:
       matchLabels:
        type: front-end
 ```
#### ReplicaSet 은 Replicaton Controller와 다르게 Selector 지정이 필요함
   
  - replicaset 생성
    ```
    $ kubectl create -f replicaset-definition.yaml
    ```
  - replicaset 리스트업
    ```
    $ kubectl get replicaset
    ```
  - replicaset에 의해 생성된 pod들은 RC의 이름이 포함
    ```
    $ kubectl get pods
    ```
   
    ![rs1](../../images/rs1.PNG)
    
## Labels and Selectors
#### What is the deal with Labels and Selectors? Why do we label pods and objects in kubernetes?
- label을 ReplicaSet에 대한 필터로 사용할 수 있음. == 여러개의 Pod 중에서 어떤것을 Monitoring 해야하는지를 편리하게 함
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/labels.PNG>
  
## replicaset 확장 방법
- replicaset을 확장하기 위한 여러 방법
  - 첫 번째 방법은 replicaset-definition.yaml 정의 파일에서 복제본 수를 업데이트하는 것입니다. E.g replicas: 6 and then run 
 ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 6
     selector:
       matchLabels:
        type: front-end
```

  ```
  $ kubectl apply -f replicaset-definition.yaml
  ```
  - 두번째 방법 : **`kubectl scale`** 커맨드
  - **`kubectl scale`** 커맨드는 구성 file까지 수정해주지는 않는다
  ```
  $ kubectl scale --replicas=6 -f replicaset-definition.yaml
  ```
  - 세번째 방법 **`kubectl scale`** 커맨드 w/ type and name
  ```
  $ kubectl scale --replicas=6 replicaset myapp-replicaset
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rs2.PNG>

#### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/13-ReplicaSets.md
