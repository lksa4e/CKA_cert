# Resource Limits
#### Let us take a look at 3 node kubernetes cluster.
- 각 노드에는 사용 가능한 CPU, 메모리 및 디스크 리소스 세트가 존재함
- 노드에 사용 가능한 리소스가 충분하지 않으면 kubernetes는 포드 예약을 유지합니다. Pending 상태의 포드가 표시됩니다. 이벤트를 보면 Pending의 이유는 CPU가 부족한 것을 알 수 있음
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rl.PNG>
  
## Resource Requirements
- Pod가 사용하는 CPU 및 Mem을 지정할 수 있다. 이것은 **`Resource Request` for a container**. 라고 부른다
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rr.PNG>
  
- Pod definition File에서 Resource Requeset를 지정할 수 있다.

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
    labels:
      name: simple-webapp-color
  spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
      - containerPort:  8080
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rr-pod.PNG>
   
## Resources - Limits
- k8s에서는 Pod가 사용 가능한 Resource를 제한할 수 있다.
- Pod가 생성될 때의 Default Limit을 지정하려면 spec: limit: 에 -default 지정을 해주어야 한다.
  ```
  apiVersion: v1
  kind: LimitRange
  metadata:
    name: mem-limit-range
  spec:
    limits:
    - default:
        memory: 512Mi
      defaultRequest:
        memory: 256Mi
      type: Container
   ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rsl.PNG>
  
  
- You can set the resource limits in the pod definition file.
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
    labels:
      name: simple-webapp-color
  spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
      - containerPort:  8080
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
       limits:
         memory: "2Gi"
         cpu: "2"
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/rsl1.PNG>
  
#### Note: 리소스에 대한 요청 및 제한은 포드의 컨테이너별로 설정.
  
## Exceed Limtis
- what happens when a pod tries to exceed resources beyond its limits?
- 쿠버네티스는 컨테이너가 제한된 CPU 리소스 이상을 넘는 요청이 오는 경우 Throttle을 발생시켜 사용할 수 없게 한다. 
- 하지만 메모리의 경우 컨테이너가 제한된 리소스보다 더 사용하게 될 수 있다. 하지만 이 상태가 지속되면 컨테이너를 종료시킨다.
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/el.PNG>
   
  
#### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/03-Scheduling/12-Resource-Limits.md
  
