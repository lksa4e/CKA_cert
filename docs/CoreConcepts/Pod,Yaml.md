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
      
## Edit a Pod
- 아래 이외의 기존 POD 사양은 편집 할 수 없습니다.
  - spec.containers[*].image
  - spec.initContainers[*].image
  - spec.activeDeadlineSeconds
  - spec.tolerations
- 예를 들어 실행중인 포드의 환경 변수, 서비스 계정, 리소스 제한을 변경할 수 없습니다. 그러나 정말로 원한다면 두 가지 옵션이 있습니다.

### 1. pod 삭제하고 다시 만들기 (kubectl edit)
```
> kubectl edit pod <pod name> //저장하려고 하면 저장 안되고 temp 파일만 남음 그거를 복사해서 yaml파일
> kubectl delete pod webapp
> kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
```
### 2. pod 삭제하고 다시 만들기 (yaml 추출)
```
> kubectl get pod webapp -o yaml > my-new-pod.yaml
> vi my-new-pod.yaml
> kubectl delete pod webapp
> kubectl create -f my-new-pod.yaml
```
### 3. Deployment를 통한 Pod 변경 (추천)
- 포드 템플릿은 Deployment spec의 하위 항목이므로 변경 될 때마다 배포가 자동으로 삭제되고 새로운 변경 사항으로 새 포드가 생성됩니다.
```
> kubectl edit deployment my-deployment
```
