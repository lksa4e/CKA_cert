# Deployments

#### Deployment is a kubernetes object. 
- 디플로이먼트(Deployment) 는 파드와 레플리카셋(ReplicaSet)에 대한 선언적 업데이트를 제공
- 디플로이먼트(Deployment)를 생성하면 자동으로 Replicaset을 만듬
- Rolling Update, Roll back, Pause 등의 역할을 위해 사용
  
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/deployment.PNG>
  
#### Deployment 정의 파일
- ReplicaSet과 거의 유사함

```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
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
 
- Deployment 생성
  ```
  $ kubectl create -f deployment-definition.yaml
  ```
- 조회
  ```
  $ kubectl get deployment
  ```

  
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
- https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/
- https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/15-Deployments.md
