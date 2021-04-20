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
- There are 2 phases involved in configuring ConfigMaps. 
  - First, create the configMaps
  - Second, Inject then into the pod.
- There are 2 ways of creating a configmap.
  - The Imperative way
    ```
    $ kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
    $ kubectl create configmap app-config --from-file=app_config.properties (Another way)
    ```
    ![cmi](../../images/cmi.PNG)
    
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
    ![cmd1](../../images/cmd1.PNG)
    
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
   
   ![cmv](../../images/cmv.PNG)
   
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
  
   ![cmp](../../images/cmp.PNG)
   
 #### There are other ways to inject configuration variables into pod   
 - You can inject it as a **`Single Environment Variable`** 
 - You can inject it as a file in a **`Volume`**
 
   ![cmp1](../../images/cmp1.PNG)
   

# Secrets
- ConfigMaps은 설정 관련 데이터를 일반적인 텍스트 포맷을 저장한다. 하지만 Secrets은 비밀스러운 데이터나 비밀번호, 비밀키 같은 민감한 데이터를 저장하기 위해 사용한다.  
- 근본적으로 Secrets은 Configmaps와 같지만 데이터를 BASE64로 인코딩해서 저장한다는 점에서 차이가 있다.  
## Web-Mysql Application

 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/web.PNG>
 
- One way is to move the app properties/envs into a configmap. But the configmap stores data into a plain text format. It is definitely not a right place to store a password.
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
  ![web1](../../images/web1.PNG)
  
- Secrets are used to store sensitive information. They are similar to configmaps but they are stored in an encrypted format or a hashed format.

#### There are 2 steps involved with secrets
- First, Create a secret
- Second, Inject the secret into a pod.
  
  ![sec](../../images/sec.PNG)
  
#### There are 2 ways of creating a secret
- The Imperative way
  ```
  $ kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd
  $ kubectl create secret generic app-secret --from-file=app_secret.properties
  ```
  ![csi](../../images/csi.PNG)
  
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

  ![csd](../../images/csd.PNG)
  
## Encode Secrets

  ![enc](../../images/enc.PNG)
  
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
  
  ![secv](../../images/secv.PNG)
  
## Decode Secrets
- To decode secrets
  ```
  $ echo -n "bX1zcWw=" |base64 --decode
  $ echo -n "cm9vdA==" |base64 --decode
  $ echo -n "cGFzd3Jk" |base64 --decode
  ```
  ![secd](../../images/secd.PNG)
  
## Configuring secret with a pod
- To inject a secret to a pod add a new property **`envFrom`** followed by **`secretRef`** name and then create the pod-definition
  
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
  ![secp](../../images/secp.PNG)
  
#### There are other ways to inject secrets into pods.
- You can inject as **`Single ENV variable`**
- You can inject as whole secret as files in a **`Volume`**

  ![seco](../../images/seco.PNG)
  
## Secrets in pods as volume
- Each attribute in the secret is created as a file with the value of the secret as its content.
  
  ![secpv](../../images/secpv.PNG)

  

#### Additional Notes: [A Note on Secrets](https://kodekloud.com/courses/certified-kubernetes-administrator-with-practice-tests/lectures/10589148)

#### K8s Reference Docs
- https://kubernetes.io/docs/concepts/configuration/secret/
- https://kubernetes.io/docs/concepts/configuration/secret/#use-cases
- https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/
