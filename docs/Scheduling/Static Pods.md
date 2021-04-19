# Static Pods 

#### kube-apiserver없이 kubelet에 포드 정의 파일을 제공하는 방법
- Pod에 대한 정보를 저장하도록 지정된 서버의 디렉토리에서 Pod definition file을 읽도록 kubelet을 구성 할 수 있음
- kubelet은 주기적으로 directory를 읽고, Pod를 생성함 + Pod가 계속 살아있도록 보장함(파괴시 재생성, 변경시 재생성)
- directory의 파일을 삭제할 경우 Pod도 자동 삭제됨(다른 명령어를 통해 Pod를 삭제하는 방법은 없음)

## Configure Static Pod
- 지정된 디렉토리는 호스트의 모든 디렉토리가 될 수 있으며 해당 디렉토리의 위치는 서비스를 실행하는 동안 옵션으로 kubelet에 전달됩니다.
  - The option is named as **`--pod-manifest-path`**.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp.PNG>
  
## Another way to configure static pod 
- kubelet.service 파일에서 직접 옵션을 지정하는 대신 config 옵션을 사용하여 다른 구성 파일에 대한 경로를 제공하고 파일에서 directoy 경로를 staticPodPath로 정의 할 수 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp1.PNG>

## View the static pods
- To view the static pods
  ```
  $ docker ps
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp2.PNG>

#### kubelet은 두 종류의 포드 (정적 포드와 API 서버의 포드)를 동시에 만들 수 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp3.PNG>

## Static Pods - Use Case

### Static pod는 k8s Control Plane에 의존적이지 않기 때문에 Control plane component 배포 가능(controller-manager, apiserver, etcd)
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp4.PNG>

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sp5.PNG>
  
## Static Pods vs DaemonSets
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/spvsds.PNG>
  

#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/16-Static-Pods.md
