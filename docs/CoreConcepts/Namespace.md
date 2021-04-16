# Namespaces
- 쿠버네티스 클러스터 내에서의 논리적인 구분 단위
- **`Pods`**, **`Deployment`** , **`Service`** 등이 Namespace별로 구분됨
- Namespace별로 정책을 다르게 가져갈 수 있다(접근 권한 etc)
- Namespace별로 리소스의 할당량(Resource Quota)를 지정할 수 있다. (dev - CPU 100, Pro - CPU 400)
- k8s 실행 시 자동으로 만들어지는 3개의 Namespace -> kube-system, Default, kube-public

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns3.PNG>
  
- 다른 Namespace에 있는 pod 조회하기
  ```
  $ kubectl get pods --namespace=kube-system
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns8.PNG>
  
- pod definition file에 namespace를 지정하지 않으면 default namespace에 자동으로 pod를 생성한다.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
     app: myapp
     type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
 ```
  ```
  $ kubectl create -f pod-definition.yaml
  ```
- 다른 Namespace에 생성하고 싶으면 **`--namespace`** 옵션을 사용
  ```
  $ kubectl create -f pod-definition.yaml --namespace=dev
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns9.PNG>

- **`--namespace`** 커맨드를 사용하지 않고 다른 Namespace에 사용하는 방법 -> pod-definition file에 namespace 항목 추가하기(metadata 아래)
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
  labels:
     app: myapp
     type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
 ```
  
  <https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns10.PNG>
  
- Namespace definition file(namespace-dev.yaml)
```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

  ```
  $ kubectl create -f namespace-dev.yaml
  ```
  다른 방법
  ```
  $ kubectl create namespace dev
  ```
  <https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns11.PNG>
  
- 별도의 command 없이 다른 Namespace를 지속적으로 사용하기 위한 방법 
  ```
  $ kubectl config set-context $(kubectl config current-context) --namespace=dev
  ```
- 모든 Namespace의 pod 출력하기
  ```
  $ kubectl get pods --all-namespaces
  ```
  
- Namespace의 리소스를 제한하기 위해서 **`ResourceQuota`** 를 사용한다. 아래는 definition file.
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
  ```
  $ kubectl create -f compute-quota.yaml
  ```
  <https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ns13.PNG>
  
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces-walkthrough/
- https://kubernetes.io/docs/tasks/administer-cluster/namespaces/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/
- https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/
  
참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/17-Namespaces.md
  
