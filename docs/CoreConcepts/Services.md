## Services
- Kubernetes Services는 애플리케이션 내부와 외부의 다양한 구성 요소 간의 통신을 가능하게 함

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srv1.PNG>
  

## External Communication

- **`외부 유저`** 가 **`web page`** 에 접속하는 방법은?

  - 노드 안에서 (예상대로 애플리케이션에 도달 할 수 있음)
  
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srv2.PNG>
    
  - 바깥쪽에서 (중간에 무언가가 없으면 응용 프로그램에 도달하지 못함)
  
    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srv3.PNG>
   
    
 ## Service Types
 
 #### kubernetes의 3가지 Service type
 
   <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srv-types.PNG>
 
 ## 1. NodePort
    - 클러스터 IP로만 접근이 가능한것이 아니라,IP와 포트를 통해서도 Node 내부 Pod에 접근이 가능하게 된다.
    - 가질 수 있는 port area는 30000~32767까지
    - 필수 영역은 port, target port는 입력 안할시 port와 동일하고, nodePort는 입력 안할 시 가능한 것중 랜덤으로 부여
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: myapp-service
      spec:
       types: NodePort
       ports:
       - targetPort: 80
         port: 80
         nodePort: 30008
      ```
     <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvnp.PNG>
      
      #### To connect the service to the pod
      - Service만 생성하고 Pod에 연결하지 않으면 작동하지 않음
      - Selector와 Label을 지정해주어야만 실제 pod와 연결할 수 있음
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: myapp-service
      spec:
       types: NodePort
       ports:
       - targetPort: 80
         port: 80
         nodePort: 30008
       selector:
         app: myapp
         type: front-end
       ```

    <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvnp1.PNG>
      
      #### To create the service
      ```
      $ kubectl create -f service-definition.yaml
      ```
      
      #### To list the services
      ```
      $ kubectl get services
      ```
      
      #### 웹 브라우저 없이 CLI로 어플리케이션 access
      ```
      $ curl http://192.168.1.2:30008
      ```
      
      <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvnp2.PNG>

      #### A service with multiple pods
      
      <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvnp3.PNG>
      
      #### Pods 들이 여러 Node에 분산되어 있을 때
     - k8s 에서 알아서 Cluster를 Service로 묶어주므로 별다른 구성 없이 동일 Port 통해서 access 가능
      <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvnp4.PNG>
     
            
 ## 2. ClusterIP
    - 이 경우 서비스는 클러스터 내에 **`가상 IP`** 를 생성하여 프런트 엔드 서버 세트와 같은 다른 서비스와 백엔드 서버 세트 간의 통신을 가능하게합니다.
    - 디폴트 설정으로, 서비스에 클러스터 IP (내부 IP)를 할당한다. 쿠버네티스 클러스터 내에서는 이 서비스에 접근이 가능하지만, 클러스터 외부에서는 외부 IP 를 할당  받지 못했기 때문에, 접근이 불가능하다
    
<https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvc1.PNG>

#### 이러한 서비스 또는 계층 간의 연결을 설정하는 올바른 방법은 무엇입니까?  
- kubernetes 서비스는 포드를 그룹화하고 그룹의 포드에 액세스 할 수있는 단일 인터페이스를 제공하는 데 도움이 될 수 있습니다.

 <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvc2.PNG>
  
#### To create a service of type ClusterIP
```
apiVersion: v1
kind: Service
metadata:
 name: back-end
spec:
 types: ClusterIP
 ports:
 - targetPort: 80
   port: 80
 selector:
   app: myapp
   type: back-end
```
```
$ kubectl create -f service-definition.yaml
```

#### To list the services
```
$ kubectl get services
```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/srvc3.PNG>

## 3. LoadBalancer
    - Where the service provisions a **`loadbalancer`** for our application in supported cloud providers.
    - 보통 클라우드 벤더에서 제공하는 설정 방식으로, 외부 IP 를 가지고 있는 로드밸런서를 할당한다. 외부 IP를 가지고 있기  때문에, 클러스터 외부에서 접근이 가능하다.


    
K8s Reference Docs:
- https://kubernetes.io/docs/concepts/services-networking/service/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/02-Core-Concepts/19-Services.md
