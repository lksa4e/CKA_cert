# DaemonSets

#### DaemonSet은 여러 포드 인스턴스를 배포하는 데 도움이 되는 Replica Set와 같습니다. 
- 하지만 POD의 복사본을 클러스터의 각 노드마다 1개씩 유지될 수 있도록 한다는 점이 다르다
- 새로운 노드가 추가된다면 자동으로 Daemonsets의 리플리카가 생성된다. 그리고 노드가 삭제되면 해당 노드의 Daemonsets POD도 삭제한다
- 쿠버네티스 v1.12까지는 DaemonSet의 POD에는 nodeName 속성에 각 노드가 명시 되었지만 그 이후 버전부터는 NodeAffinity를 사용하고 있다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds.PNG>
  
## DaemonSets - UseCases

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds-uc.PNG>
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds-uc-kp.PNG>
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds-ucn.PNG>
  
## DaemonSets - Definition
- DaemonSet을 만드는 과정은 ReplicaSet를 만드는 것과 유사함
- For DaemonSets, we start with apiVersion, kind as **`DaemonSets`** instead of **`ReplicaSet`**, metadata and spec. 
  ```
  apiVersion: apps/v1
  kind: Replicaset
  metadata:
    name: monitoring-daemon
    labels:
      app: nginx
  spec:
    selector:
      matchLabels:
        app: monitoring-agent
    template:
      metadata:
       labels:
         app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
  ```
  
  ```
  apiVersion: apps/v1
  kind: DaemonSet
  metadata:
    name: monitoring-daemon
    labels:
      app: nginx
  spec:
    selector:
      matchLabels:
        app: monitoring-agent
    template:
      metadata:
       labels:
         app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/dsd.PNG>
  
- To create a daemonset from a definition file
  ```
  $ kubectl create -f daemon-set-definition.yaml
  ```

## View DaemonSets
- To list daemonsets
  ```
  $ kubectl get daemonsets
  ```
- For more details of the daemonsets
  ```
  $ kubectl describe daemonsets monitoring-daemon
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds1.PNG>
  
## How DaemonSets Works

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ds2.PNG>

#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec
