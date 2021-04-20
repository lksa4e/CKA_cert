# Configure Environment Variables In Applications

#### ENV variables in Docker
```
$ docker run -e APP_COLOR=pink simple-webapp-color
```

#### ENV variables in kubernetes 
- 환경변수 지정을 위해 **`env`** 항목을 pod definition file에 추가.
- `ENV` 속성은 배열 타입이다. `ENV` 하위 값은 Name과 Value 속성으로 구성된다.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
  spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
     - containerPort: 8080
     env:
     - name: APP_COLOR
       value: pink
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/env.PNG>
  
-  환경변수를 지정하는 다른 방법은 **`ConfigMaps`** and **`Secrets`** 을 사용하는 방법이 있다.
-  이 방식을 사용할 때는 Value를 명시하는 대신 `valueFrom`을 사용해서 ConfigMaps과Secret을 선택한다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cms.PNG>
  
# Configure ConfigMaps in Applications

## ConfigMaps
- ConfigMaps는 쿠버네티스 상에 설정 관련 데이터를 Key-Value 형태로 저장할 때 사용하는 오브젝트이다
- ConfigMaps 구성의 2가지 단계. 
  - 1. configMaps 생성
  - 2. configMaps를 Pod에 넣기
- configmap을 만드는 2가지 방법.
  - The Imperative way
    ```
    $ kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
    $ kubectl create configmap app-config --from-file=app_config.properties (Another way)
    ```
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cmi.PNG>
    
  - The Declarative way
    
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: app-config
    data:
     APP_COLOR: blue
     APP_MODE: prod
    ```
    ```
    Create a config map definition file and run the 'kubectl create` command to deploy it.
    $ kubectl create -f config-map.yaml
    ```
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cmd1.PNG>
    
 ## View ConfigMaps
 - To view configMaps
   ```
   $ kubectl get configmaps (or)
   $ kubectl get cm
   ```
 - To describe configmaps
   ```
   $ kubectl describe configmaps
   ```
   
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cmv.PNG>
   
 ## ConfigMap in Pods
 - Inject configmap in pod
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: simple-webapp-color
   spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - configMapRef:
          name: app-config
   ```
   ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     APP_COLOR: blue
     APP_MODE: prod
   ```
   ```
   $ kubectl create -f pod-definition.yaml
   ```
  
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cmp.PNG>
   
 #### 구성 변수를 포드에 삽입하는 다른 방법이 있습니다  
 - **`Single Environment Variable`** 로 삽입
 - **`Volume`** 에 파일로 삽입
 
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/cmp1.PNG>
   

# Secrets
- ConfigMaps은 설정 관련 데이터를 일반적인 텍스트 포맷을 저장한다. 하지만 Secrets은 비밀스러운 데이터나 비밀번호, 비밀키 같은 민감한 데이터를 저장하기 위해 사용한다.  
- 근본적으로 Secrets은 Configmaps와 같지만 데이터를 BASE64로 인코딩해서 저장한다는 점에서 차이가 있다.  
- Secrets을 매니페스트 파일로 정의해서 생성할 때는 반드시 데이터를 BASE64로 해시한 뒤 입력해야 한다
## Web-Mysql Application
 
- 한 가지 방법은 앱 속성 / env를 configmap으로 이동하는 것입니다. 그러나 configmap은 데이터를 일반 텍스트 형식으로 저장합니다. 확실히 암호를 저장하기에 적합한 장소가 아닙니다.
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
   name: app-config
  data:
    DB_Host: mysql
    DB_User: root
    DB_Password: paswrd
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/web1.PNG>

#### There are 2 steps involved with secrets
- First, Create a secret
- Second, Inject the secret into a pod.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sec.PNG>
  
#### There are 2 ways of creating a secret
- The Imperative way
  ```
  $ kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd
  $ kubectl create secret generic app-secret --from-file=app_secret.properties
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csi.PNG>
  
- The Declarative way
  ```
  Generate a hash value of the password and pass it to secret-data.yaml definition value as a value to DB_Password varaible.
  $ echo -n "mysql" |base64
  $ echo -n "root" |base64
  $ echo -n "paswrd"|base64
  ```
  
  Create a secret definition file and run `kubectl create` to deploy it
  ```
  apiVersion: v1
  kind: Secret
  metadata:
   name: app-secret
  data:
    DB_Host: bX1zcWw=
    DB_User: cm9vdA==
    DB_Password: cGFzd3Jk
  ```
  ```
  $ kubectl create -f secret-data.yaml
  ```

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/csd.PNG>
  
## Encode Secrets

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/enc.PNG>
  
## View Secrets
- To view secrets
  ```
  $ kubectl get secrets
  ```
- To describe secret
  ```
  $ kubectl describe secret
  ```
- To view the values of the secret
  ```
  $ kubectl get secret app-secret -o yaml
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/secv.PNG>
  
## Decode Secrets
- To decode secrets
  ```
  $ echo -n "bX1zcWw=" |base64 --decode
  $ echo -n "cm9vdA==" |base64 --decode
  $ echo -n "cGFzd3Jk" |base64 --decode
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/secd.PNG>
  
## Configuring secret with a pod
- Secret을 Pod에 삽입하려면 새 속성 **`envFrom`** 다음에 **`secretRef`** 이름을 추가 한 다음 포드 정의를 만듭니다.
  
  ```
  apiVersion: v1
  kind: Secret
  metadata:
   name: app-secret
  data:
    DB_Host: bX1zcWw=
    DB_User: cm9vdA==
    DB_Password: cGFzd3Jk
  ```
  ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: simple-webapp-color
   spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - secretRef:
          name: app-secret
   ```
  ```
  $ kubectl create -f pod-definition.yaml
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/secp.PNG>
  
#### There are other ways to inject secrets into pods.
- **`Single ENV variable`** 로 삽입
- **`Volume`** 에 파일처럼 전체 비밀을 삽입 할 수 있습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/seco.PNG>
  
## Secrets in pods as volume
- 비밀의 각 속성은 비밀의 값을 내용으로하는 파일로 생성됩니다.
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/secpv.PNG>


#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/concepts/configuration/secret/#use-cases
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/05-Application-Lifecycle-Management/10.Secrets.md
