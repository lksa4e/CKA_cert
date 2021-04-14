# Kube API Server

#### Kube-apiserver is the primary component in kubernetes.
- Kube-apiserver는 요청을 **`인증`**, **`검증`** , 데이터를 ETCD key-value 저장소에서 **`검색`** 및 **`업데이트`** 하는 역할을 함. 실제로 kube-apiserver는 etcd 데이터 저장소와 직접 상호 작용하는 유일한 구성 요소. kube-scheduler, kube-controller-manager 및 kubelet과 같은 다른 구성 요소는 API-Server를 사용하여 해당 영역의 클러스터에서 업데이트함.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/post.PNG>
  
## Installing kube-apiserver

-  **`kubeadm`** 도구를 사용하여 kube-apiserver를 부트스트랩 하는 경우 이를 알 필요가 없지만 hardway를 사용하여 설정하는 경우 kubernetes 릴리스 페이지에서 kube-apiserver를 바이너리로 사용할 수 있습니다..
  - kube-apiserver v1.13.0 binary
    ```
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
    ```
 
## View kube-apiserver - Kubeadm
- kubeadm은 kube-apiserver를 마스터 노드의 kube-system 네임 스페이스에 포드로 배포함.
  ```
  $ kubectl get pods -n kube-system
  ```
   
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/kube-apiserver1.PNG>
   
## View kube-apiserver options - Kubeadm
-  **`/etc/kubernetes/manifests/kube-apiserver.yaml`** 에 있는 POD 정의 파일에서 옵션을 볼 수 있음
  ```
  $ cat /etc/kubernetes/manifests/kube-apiserver.yaml
  ```
   
## View kube-apiserver options - Manual
- kubeadm을 사용하지 않은 경우 kube-apiserver.service 파일에서 옵션을 확인할 수 있음
  ```
  $ cat /etc/systemd/system/kube-apiserver.service
  ```  
- 마스터 노드에 프로세스를 나열하고 kube-apiserver를 검색하여 실행중인 프로세스 및 영향 옵션을 볼 수도 있음
  ```
  $ ps -aux |grep kube-apiserver
  ```

K8s Reference Docs:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/concepts/overview/kubernetes-api/
- https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/
- https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/05-Kube-API-Server.md#kube-apiserver-is-the-primary-component-in-kubernetes
