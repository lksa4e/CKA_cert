# Multiple Schedulers
- 쿠버네티스는 확장성이 뛰어나다. 사용자가 직접 스케줄러 프로그램을 만들어서 기본 스케줄러와 함께 실행되도록 할 수 있다.  
- kubeadm으로 클러스터를 구성했다면 `/etc/kubenetes/manifests/` 폴더에 Static POD로 실행될 kube-scheduler 매니페스트가 존재한다.  

- 가장 쉽게 커스텀 스케줄러를 만들기 위한 방법은 기존 kube-scheduler Static POD 매니페스트를 복사해서 POD의 name과 kube-scheduler 커맨드 옵션을 추가하는 방법이다.  

- 여러 마스터 노드에서 각 실행되는 스케줄러에 리더 선정을 위해 `--leader-elect` 옵션을 사용할 수 있다.  
  - `--leader-elect` 옵션을 통해 여러 마스터 노드에 동일한 스케줄러가 실행 중일 때 반드시 1개만 리더로 활성화될 수 있도록 한다.  
  - 다수의 마스터 노드를 가지고 있지 않은 경우에 (1개의 Master Node)여러개의 Scheduler를 사용할 때는 `--leader-elect` 옵션을 False로 해주어야 한다.

- 다수의 Master Node를 가지고 있는 경우에 Default Scheduler와 Custom Scheduler를 구분해주기 위해 `--lock-object-name` 옵션을 사용한다.
- `--lock-object-name` 옵션은 리더 선택 프로세스에서 Custom Scheduler가 제외되도록 한다.  

- 커스텀 스케줄러는 POD나 Deployment definition file에서 정의할 수 있다.  
   특정 스케줄러만을 지정하려면 POD spec에 `schedulerName` 속성을 이용하면 된다. 

## Custom Schedulers
- kubernetes 클러스터는 동시에 여러 스케줄러를 예약 할 수 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ms.PNG>
  
## Deploy additional scheduler
- Download the binary
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/das.PNG>
  
## Deploy additional scheduler - kubeadm
- command section은 스케쥴러를 실행시킬 때의 option과 관련
   
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/dask.PNG>
  
  - To create a scheduler pod
    ```
    $ kubectl create -f my-custom-scheduler.yaml
    ```
  
## View Schedulers
- To list the scheduler pods
  ```
  $ kubectl get pods -n kube-system
  ```

## Use the Custom Scheduler
- Pod definition file을 만들고 **`schedulerName`** 이라는 새 섹션을 추가하고 새 스케줄러의 이름을 지정합니다
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - image: nginx
      name: nginx
    schedulerName: my-custom-scheduler
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cs.png>
  
- To create a pod definition
  ```
  $ kubectl create -f pod-definition.yaml
  ```
- To list pods
  ```
  $ kubectl get pods
  ```

## View Events
- To view events
- 어떤 스케쥴러가 Pod를 배치했는지 확인
  ```
  $ kubectl get events
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cs1.PNG>
  
## View Scheduler Logs
- To view scheduler logs
  ```
  $ kubectl logs my-custom-scheduler -n kube-system
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cs2.PNG>
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/18-Multiple-Schedulers.md
  
