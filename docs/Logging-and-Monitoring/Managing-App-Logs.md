# Managing Application Logs
#### Let us start with logging in docker

<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ld.PNG>
 
<img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/ld1.PNG>
 
#### Logs - Kubernetes
```
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
```
 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/logs-k8s.png>
 
- To view the logs
  ```
  $ kubectl logs -f event-simulator-pod
  ```
- Pod에 여러 컨테이너가있는 경우 명령에 명시 적으로 컨테이너 이름을 지정해야합니다.
  ```
  $ kubectl logs -f <pod-name> <container-name>
  $ kubectl logs -f even-simulator-pod event-simulator
  ```

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/logs1.PNG>
  
#### K8s Reference Docs
- https://kubernetes.io/blog/2015/06/cluster-level-logging-with-kubernetes/
 
