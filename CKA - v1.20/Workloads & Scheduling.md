# Understand deployments and how to perform rolling update and rollbacks
## Agenda 
1. Understand deployments and how to perform rolling update and rollbacks
2. Use ConfigMaps and Secrets to configure applications
3. Know how to scale applications
4. Understand the primitives used to create robust, self-healing, application deployments
5. Understand how resource limits can affect Pod scheduling
6. Awareness of manifest management and common templating tools

deployment - https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ <br>
Configmap - https://kubernetes.io/docs/concepts/configuration/configmap/ <br>
Secret - https://kubernetes.io/docs/concepts/configuration/secret/ <br>
Scale Application - https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#scaling-your-application <br>
robust, self-healing, application deployment - https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/ , https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ <br>
Resource Limit - https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/ <br>
Manifest management, common templating tool - https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/ , https://kubernetes.io/docs/tasks/manage-kubernetes-objects/ <br>

Deployments는 Replication Controller를 대체하기 위한 것입니다. 동일한 복제 기능(Replica Sets를 통한)과 변경 사항을 롤아웃하고 필요한 경우 롤백하는 기능도 제공합니다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```

`deployments` 를 활용하는 주된 이유는 하나의 관리 단위인 'Deployment' 개체를 통해 동일한 Pod를 여러 개 관리하기 위해서입니다. 
변경이 필요한 경우 개별 포드가 아닌 `deployment` 객체에 이를 적용합니다. `deployments` 의 선언적 특성 때문에, 쿠버네티스는 원하는 상태와 작동 중인 상태 사이의 모든 변경 사항을 비교하고 그에 따라 수정합니다. 

기존 deployment를 업데이트하는 방법은 2가지 Option이 있습니다:

*   Rolling Update
*   Recreate

Rolling Update는 Deployment의 컨테이너를 새 이미지로 만든 컨테이너와 교환합니다. (새로운 버전 하나 만들고 기존꺼 하나씩 지우는 방식)

애플리케이션이 서로 다른 포드(애플리케이션 버전)의 혼합을 지원하는 경우 롤링 업데이트를 사용합니다(한번에 모두 버전업 하는게 아니라 버전 다른게 섞여도 상관없는 경우). 
이 방법은 서비스 중단 시간(Downtime)이 없지만 Deployment를 요청된 버전으로 업데이트하는 데 더 오래 걸립니다. 
업데이트가 모두 완료될때까지 Pod는 이전 버전과 새 버전이 공존합니다.

recreation 방식은 기존의 모든 포드를 삭제한 후 한번에 새로운 포드로 바꿉니다. 이 방법은 Downtime이 존재합니다. (“big bang” 접근법)

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.15
        ports:
        - containerPort: 80
```
Imperative Way - `kubectl set image deployment/frontend www=nginx:v2 --record` --> frontend deployment의 www 컨테이너의 image를 nginx:v2로 변경
Declarative Way - ` kubectl apply -f updateddeployment.yaml --record=true`  --> file의 변경사항 적용

Followed by the following:

```shell
kubectl rollout status deployment/nginx-deployment

Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 3 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 4 out of 5 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 4 of 5 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
```

또한 kubectl rollout history 커맨드를 사용하여 배포의 revision 기록을 볼 수 있습니다.


```shell
kubectl rollout history deployment/nginx-deployment

deployment.extensions/nginx-deployment
REVISION  CHANGE-CAUSE
1     	<none>
2     	<none>
4     	<none>
5     	kubectl apply --filename=updateddeployment.yaml --record=true
```
또는 imperative way로 작업을 수행할 수도 있습니다:

```shell
kubectl set image deployments/nginx-deployment nginx=nginx:1.9.1 --record

deployment.extensions/nginx-deployment image updated
deployment.extensions/nginx-deployment image updated
```

## Rollback

이전 버전으로 롤백:

```shell
kubectl rollout undo deployment/nginx-deployment
```

특정 버전으로 롤백:

```shell
kubectl rollout undo deployment/nginx-deployment --to-revision 5
```

`revision` 기록확인? : `kubectl rollout history deployment/nginx-deployment`

# ConfigMaps 및 Secrets를 사용하여 애플리케이션 구성

Configmap은 Pod manifest에서 구성을 분리하는 방법입니다. 분명히 첫 번째 단계는 pod를 사용하기 전에 구성 맵을 만드는 것입니다.
Pod와 Configmap은 동일한 Namespace에 있어야 합니다.

```shell
kubectl create configmap <map-name> <data-source>
```

"Map-name"은이 특정 맵에 부여하는 임의의 이름이며 "data-source"는 구성 맵에있는 키-값 쌍에 해당합니다.

```shell
kubectl create configmap vt-cm --from-literal=blog=virtualthoughts.co.uk
```

```shell
kubectl describe configmap vt-cm
Name:         vt-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
blog:
----
virtualthoughts.co.uk
```

Pod에서 Configmap을 참조하기 위해서 Pod definition file에서 선언합니다:

Configmap은 'Volumes' 또는 'Environment'로 마운트 할 수 있습니다. 아래 예는 후자(환경 변수)를 활용합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: config-test-pod
spec:
 containers:
 - name: test-container
   image: busybox
   command: [ "/bin/sh", "-c", "env" ]
   env:
     - name: BLOG_NAME
       valueFrom:
         configMapKeyRef:
           name: vt-cm
           key: blog
 restartPolicy: Never
```

위의 포드는 환경 변수를 출력하므로(command: `[ "/bin/sh", "-c", "env" ]`) 포드에서 로그를 추출하여 구성 맵을 활용하는지 확인할 수 있습니다:

```shell
kubectl logs config-test-pod | grep "BLOG_NAME="
...
BLOG_NAME=virtualthoughts.co.uk
...
```

#  scale applications

지속적으로 더 많은 개별 포드를 추가하는 것은 애플리케이션 확장을위한 지속 가능한 모델이 아닙니다. 
대규모 애플리케이션을 용이하게 하려면 ReplicaSet 또는 Deployment와 같은 더 높은 수준의 구성을 활용해야합니다. 
`Deployment`는 기본 포드를 관리하기 위한 단일 관리 단위를 제공합니다. `deployment` 객체를 확장하여 `pods`의 수를 늘릴 수 있습니다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 name: nginx-deployment
spec:
 replicas: 5
 template:
   metadata:
     labels:
       app: nginx-frontend
   spec:
     containers:
     - name: nginx
       image: nginx:1.14
       ports:
       - containerPort: 80
```

이를 확장하려면 "replicas"필드를 수정하여 yaml 파일을 수정하고 Deployment를 확장/축소하거나 즉석에서 수정할 수 있습니다:

```shell
kubectl scale deployment nginx-deployment --replicas 10
```

# Understand the primitives used to create robust, self-healing, application deployments

Deployment는 조정 루프(reconciliation loop)를 사용하여 배포된 포드 수가 매니페스트에 정의된 것과 일치하는지 확인함으로써 이 작업을 용이하게 합니다.
내부적으로 Deployment는 이 기능을 주로 담당하는 ReplicaSets를 활용합니다.

Stateful Sets는 Deployment와 유사합니다. 예를 들어 일련의 Pod의 배포 및 확장을 관리합니다. 
하지만 deployments의 기능에 더해서 Pod의 순서 및 고유성에 대한 보장도 제공합니다. StatefulSet는 각 포드에 대해 고정 ID를 유지합니다.
이러한 포드는 동일한 사양에서 생성되지만 상호 교환 할 수는 없습니다 : 각각은 다시 스케줄링 할 경우 유지되는 영구 식별자를 가지고 있습니다.

StatefulSet은 다음 중 하나 이상이 필요한 애플리케이션에 유용합니다.

*   Stable, unique network identifiers.
*   Stable, persistent storage.
*   Ordered, graceful deployment and scaling.
*   Ordered, automated rolling updates.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
 name: nginx-statefulset
spec:
 selector:
   matchLabels:
     app: vt-nginx
 serviceName: "nginx"
 replicas: 2
 template:
   metadata:
     labels:
       app: vt-nginx
   spec:
     containers:
     - name: vt-nginx
       image: nginx:1.7.9
       ports:
       - containerPort: 80
```

# Understand how resource limits can affect Pod scheduling

Namespace 레벨에서 리소스 제한을 정의할 수 있습니다. 특히 멀티 테넌시 환경에서 유용하며, Pod가 허용 된 것보다 많은 리소스를 소비하여 전체 환경에 해로운 영향을 미칠 수 있는 것을 방지하는 메커니즘을 제공합니다. 

We can define the following:

Default memory / CPU **requests & limits** for a namespace

Minimum and Maximum memory / CPU **constraints** for a namespace

Memory/CPU **Quotas** for a namespace

## Default Requests and Limits

컨테이너가 default Request/Limit 값이 존재하는 네임스페이스에서 생성되고 컨테이너 매니페스트에서 명시적으로 정의하지 않은 경우 네임스페이스에 선언된 default 값을 상속합니다

Note, 만약 컨테이너에서 memory/CPU limit만 정의하고 request를 정의하지 않는다면, Kubernetes는 request와 limit을 동일하게 정의합니다.

## Minimum / Maximum Constraints

팟(Pod)이 제약조건을 만족하지 않으면 스케쥴링되지 않습니다.

## Quotas

_namespace_에서 전체적으로 사용할 수있는 _total_ CPU / 메모리 양을 제어합니다.

Example: 네임 스페이스에 정의 된 것보다 더 많은 메모리를 요청하는 포드를 예약

Create a namespace:
```shell
kubectl create namespace tenant-mem-limited
```
Create a YAML manifest to limit resources:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: tenant-max-mem
  namespace: tenant-mem-limited
spec:
  limits:
  - max:
      memory: 250Mi
    type: Container
```

앞서 언급한 namespace에 적용: 

```shell
kubectl apply -f maxmem.yaml`
```

Limit을 초과하는 메모리 Request를 가지는 포드:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: too-much-memory
  namespace: tenant-mem-limited 
spec:
  containers:
  - name: too-much-mem
    image: nginx
    resources:
      requests:
        memory: "300Mi"
```

위를 실행하면 다음과 같은 결과가 나타납니다.:

```shell
The Pod "too-much-memory" is invalid: spec.containers[0].resources.requests: Invalid value: "300Mi": must be less than or equal to memory limit
```

네임 스페이스의 포드 제한을 250MiB로 정의 했으므로 300MiB에 대한 요청이 실패합니다.

#  Awareness of manifest management and common templating tools

## Kustomize

Kustomize는 네이티브 형식 (Yaml)의 Kubernetes 매니페스트를 위한 템플릿 도구입니다. 
원시 YAML 파일로 작업 할 때 일반적으로 생성되는 리소스를 식별하는 여러 파일이 포함 된 디렉토리가 있습니다. 
시작하려면 매니페스트가 포함 된 디렉토리가 있어야합니다:

```shell
/home/david/app/base
total 16
drwxrwxr-x  2 david david 4096 Feb  9 11:44 .
drwxr-xr-x 27 david david 4096 Feb  9 11:44 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

이것은 우리의 `base`를 형성할 것입니다 - 우리는 이것 기반에서 오버레이 형태로 사용자 정의를 추가 할 것입니다. 
먼저`kustomize` 파일이 필요합니다. `kustomize create --autodetect`로 만들 수 있습니다.

현재 디렉토리에 kustomization.yaml 생성:

```shell
total 20
drwxrwxr-x  2 david david 4096 Feb  9 11:47 .
drwxr-xr-x 27 david david 4096 Feb  9 11:47 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  108 Feb  9 11:47 kustomization.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

The contents being:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
```

## Variants and Overlays

* variant - `base` 에서 구성의 차이
* overlay - Composes variants together(함께 변형 구성)

예를 들어, 이 구성에 기반을 두고 있지만 추가 사용자 지정이 있는 다른 환경(Prod and dev)에 대한 매니페스트를 생성하려고 했습니다. 
이 예에서는 단일 오버레이로 캡슐화된 `dev` variant(변형)을 생성합니다

```shell
mkdir -p overlays/{dev,prod}
cd overlays/dev 
```
베이스를 지정하는 Kustomization 객체를 생성하여 시작합니다. (this will create `kustomization.yaml`) :

```shell
kustomize create --resources ../../base
```

이 예에서는 개발 환경이므로 replica count를 1로 변경하고 싶습니다. `dev` 디렉토리에서 다음을 포함하는 새 파일`deployment.yaml`을 만듭니다:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
```

`kustomization.yaml` 파일은`patchesStrategicMerge` 블록을 포함하도록 수정해야합니다:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
patchesStrategicMerge:
  - deployment.yaml
```
패치를 사용하여 리소스에 다양한 사용자 지정을 적용 할 수 있습니다. 
Kustomize는 `patchesStrategicMerge` 및`patchesJson6902`를 통해 다양한 패치 메커니즘을 지원합니다. `patchesStrategicMerge`는 파일 경로 목록입니다.

(기본 폴더에서) 실행하여 매니페스트를 생성하고 클러스터에 적용 할 수 있습니다:

```shell
kustomize build ./overlay/dev | kubectl apply -f -
```

이것을 실행하면 `base`에 정의된 것이 아닌 우리가 적용한 사용자 지정이 적용되어 Deployment 객체에 1 개의 포드만 생성됩니다. prod 또는 임의의 수의 환경에서도 동일한 작업을 수행 할 수 있습니다.

## Helm

Helm은 Linux 세계에서`apt` 또는`yum`과 동의어입니다. 사실상 Kubernetes의 패키지 관리자입니다.
Helm에서 "Packages"는 'Charts'라고 불리우며 사용자가 원하는 값으로 커스터마이징 할 수 있습니다.

시험에서 누군가가 처음부터 helm chart를 작성해야 할 것 같지는 않지만 작동 방식을 이해하는 것이 좋습니다.

## Helm Repos

Repos는 Helm Chart가 저장되는 곳입니다. 일반적으로 저장소에는 선택할 수있는 여러 차트가 포함됩니다. Helm은 CLI 클라이언트로 관리 할 수 있으며 다음을 실행하여 저장소를 추가 할 수 있습니다:

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

To list the packages from this repo:

```shell
helm search repo bitnami
```

To install a package from this repo:

```shell
helm install my-release bitnami/mariadb
```

여기서`my-release`는이 애플리케이션의 설치된 인스턴스를 식별하는 문자열입니다

Parameters는 chart가 어떻게 구성되었는지에 따라 다릅니다.

이러한 값은 저장소의 해당`values.yaml` 파일에 캡슐화됩니다. 인스턴스를 채우고 적용 할 수 있습니다:

```shell
helm install -f https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mariadb/values.yaml my-release bitnami/mariadb
```

Alternatively, variables can be declared by using `--set`, such as:

```shell
helm install my-release --set auth.rootPassword=secretpassword bitnami/mariadb
```

참고 : https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/02-Workloads%20%26%20Scheduling.md
