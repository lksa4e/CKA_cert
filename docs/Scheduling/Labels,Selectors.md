# Labels and Selectors

#### labels과 selector는 오브젝트를 선택하고 그룹화 하기 위해 사용된다.

▶︎ `env`라벨이 `prod`인 모든 오브젝트 찾기
```shell
$ kubectl get all -l env=prod
```

▶︎ 여러 라벨로 POD 찾기
```shell
$ kubectl get po -l env=prod,bu=finance,tier=frontend
```
  
#### 레이블은 각 항목에 연결된 속성입니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/labels-ckc.PNG>
  
#### Selectors 이러한 항목을 필터링하는 데 도움이 됩니다
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sl.PNG>
  
kubernetes에서 Labels, Selector가 사용되는 방법  
How do you specify labels?
   ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: simple-webapp
     labels:
       app: App1
       function: Front-end
    spec:
     containers:
     - name: simple-webapp
       image: simple-webapp
       ports:
       - containerPort: 8080
   ```
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/lpod.PNG>
 
포드가 생성된 후, 레이블이있는 포드를 선택하려면 아래 명령을 실행합니다.
```
$ kubectl get pods --selector app=App1
```

Kubernetes는 라벨을 사용하여 서로 다른 개체를 함께 연결합니다.
   ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      labels:
        app: App1
        function: Front-end
    spec:
     replicas: 3
     selector:
       matchLabels:
        app: App1
    template:
      metadata:
        labels:
          app: App1
          function: Front-end
      spec:
        containers:
        - name: simple-webapp
          image: simple-webapp   
   ```

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/lrs.PNG>

For services
 
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: my-service
      spec:
       selector:
         app: App1
       ports:
       - protocol: TCP
         port: 80
         targetPort: 9376 
       ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/lrs1.PNG>
  
## Annotations
- annotations는 상세 정보를 표현하는 목적으로 사용된다.
    ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      labels:
        app: App1
        function: Front-end
      annotations:
         buildversion: 1.34
    spec:
     replicas: 3
     selector:
       matchLabels:
        app: App1
    template:
      metadata:
        labels:
          app: App1
          function: Front-end
      spec:
        containers:
        - name: simple-webapp
          image: simple-webapp   
    ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/annotations.PNG>

K8s Reference Docs:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/04-Labels-and-Selectors.md
https://github.com/jonnung/cka-practice/blob/master/2-scheduling.md#kube-scheduler-%EA%B0%80-%EC%8B%A4%ED%96%89%EB%90%98%EA%B3%A0-%EC%9E%88%EC%A7%80-%EC%95%8A%EC%9D%84-%EB%95%8C-pod%EB%A5%BC-%EC%A7%81%EC%A0%91-%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81-%ED%95%98%EA%B8%B0
