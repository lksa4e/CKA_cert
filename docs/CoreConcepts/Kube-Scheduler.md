# Kube Scheduler

#### kube-scheduler는 노드에 Pod를 스케쥴링하는 역할  
- kube-scheduler 어떤 Pod가 어떤 노드로 이동하는지 결정하는 작업만 담당. 실제로 노드에 포드를 배치하지 않음. 실제 Pod를 배치하는 일은 **`kubelet`** 의 일

#### Scheduler 동작 방법
1. Pod를 Node에 배치할 때 불가능한 Node들을 제외 (Filtering)
2. 우선순위에 의해 랭크를 매김 (ex. Pod 배치하고 난 뒤에 남은 잔여 CPU) 
3. 물론 Customize 가능
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kube-scheduler2.PNG>
    
## Install kube-scheduler - Manual
- kubernetes 릴리스 페이지에서 kubescheduler 바이너리를 다운로드.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
  ```
- Extract it
- Run it as a service

  
## View kube-scheduler options - kubeadm
- kubeadm를 사용하는 경우 kube-scheduler를 마스터 노드의 kube-system 네임스페이스에 Pod로 배포함
  ```
  $ kubectl get pods -n kube-system
  ```
- **`/etc/kubernetes/manifests/kube-scheduler.yaml`** 에 있는 Pod 정의 파일에서 옵션 확인
  ```
  $ cat /etc/kubernetes/manifests/kube-scheduler.yaml
  ```
  
- 마스터 노드에 프로세스를 나열하고 kube-apiserver를 검색하여 실행중인 프로세스 및 영향 옵션을 볼 수 있음
  ``` 
  $ ps -aux |grep kube-scheduler
  ```
  
  K8s Reference Docs:
  - https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/
  - https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/
  - https://kubernetes.io/docs/concepts/overview/components/
  - https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
    
참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/07-Kube-Scheduler.md
