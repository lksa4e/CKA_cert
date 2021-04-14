# Pod
- k8s에서 컨테이너를 관리하는 최소 단위
- 하나의 Pod에는 여러개의 컨테이너가 들어갈 수 있지만 같은 컨테이너가 들어가지는 않음!(Helper Container 등이 들어감)
- 동일 Pod에 속한 Container들은 Volume을 공유하고, 별도 Network 없이 서로 통신 가능

### Pod 만들기
```
$ kubectl run nginx --image nginx 
```
### Pod 리스트 출력
```
$ kubectl get pods
```

# Yaml

- k8s의 Yaml 파일은 항상 4개의 Top level 필드를 가짐(= root level properties)
  - apiVersion : k8s obeject를 생성하기 위한 k8s API version (Pod - v1 , Service - v1, ReplicaSet - apps/v1, Deployment - apps/v1)
  - kind : object의 종류
  - metadata : object의 데이터 (ex. name, labels<app,type,,>, etc)
  - spec : object 구성 정보(컨테이너,,,)

```yaml
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
      
