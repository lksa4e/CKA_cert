# Rolling Updates and Rollback
- 새로운 롤아웃은 새로운 Deployment의 수정이 생기면 revison 2라는 형태로 생성된다.
   이것은 변경을 추적할 수 있도록 도와주고 필요할 때 Deployment의 이전 버전으로 롤백할 때 유용하다.

## Rollout and Versioning in a Deployment
- 처음 deployment를 만들면 rollout을 트리거 하고, 이 rollout은 revision 1을 만든다.
- 이후에 어플리케이션이 수정되어 update가 일어나면 새로운 rollout이 트리거되고, 이 rollout이 revision 2를 만든다.
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rollv.PNG>
  
## Rollout commands
- rollout Status 조회
  ```
  $ kubectl rollout status deployment/myapp-deployment
  ```
- To see the history and revisions
  ```
  $ kubectl rollout history deployment/myapp-deployment
  ```
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rollc.PNG>
  
## Deployment Strategies
- 2가지 deployment 전략
  1. Recreate
  2. RollingUpdate (Default Strategy)
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/dst.PNG>
  
## kubectl apply
- update a deployment
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
   name: myapp-deployment
   labels:
    app: nginx
  spec:
   template:
     metadata:
       name: myap-pod
       labels:
         app: myapp
         type: front-end
     spec:
      containers:
      - name: nginx-container
        image: nginx:1.7.1
   replicas: 3
   selector:
    matchLabels:
      type: front-end       
  ```
  ```
  $ kubectl apply -f deployment-definition.yaml
  ```
- imgae update를 통한 deployment 업데이트.
  ```
  $ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ka.PNG>
  
## Recreate vs RollingUpdate
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rcrl.PNG>
  
## Upgrades

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/up.PNG>
  
## Rollback
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rb.PNG>
  
- To undo a change
  ```
  $ kubectl rollout undo deployment/myapp-deployment
  ```
  
## kubectl create
- To create a deployment
  ```
  $ kubectl create deployment nginx --image=nginx
  ```
## Summarize kubectl commands
```
$ kubectl create -f deployment-definition.yaml
$ kubectl get deployments
$ kubectl apply -f deployment-definition.yaml
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
$ kubectl rollout status deployment/myapp-deployment
$ kubectl rollout history deployment/myapp-deployment
$ kubectl rollout undo deployment/myapp-deployment
```

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sum.PNG>
 
#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment
- https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/05-Application-Lifecycle-Management/02-RollingUpdates-and-Rollback.md
