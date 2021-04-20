# Commands and Arguments in Docker
- To run a docker container
  ```
  $ docker run ubuntu
  ```
- To list running containers
  ```
  $ docker ps 
  ```
- To list all containers including that are stopped
  ```
  $ docker ps -a
  ```
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/dc.PNG>
  
#### 가상 머신과 달리 컨테이너는 운영 체제를 호스팅하기위한 것이 아닙니다.
- 컨테이너는 웹 서버, 애플리케이션 서버 또는 데이터베이스 서버 등의 인스턴스를 호스팅하는 것과 같은 특정 작업 또는 프로세스를 실행하기위한 것입니다.

#### 컨테이너를 시작하는 다른 명령을 어떻게 지정합니까?
- 한 가지 옵션은 docker run 명령에 명령을 추가하는 것입니다. 그러면 이미지에 지정된 기본 명령이 재정의됩니다.
  ```
  $ docker run ubuntu sleep 5
  ```
- 이렇게하면 컨테이너가 시작될 때 수면 프로그램을 실행하고 5 초 동안 기다린 다음 존재합니다. 그 변화를 어떻게 영구적으로 만드나요?
  
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sleep.PNG>
  
- 명령을 단순히 쉘 형식으로 또는 JSON 배열 형식으로 지정하는 여러 가지 방법이 있습니다.
 
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/sleep1.PNG>
  
- Now, build the docker image
  ```
  $ docker build -t ubuntu-sleeper .
  ```
- Run docker container
  ```
  $ docker run ubuntu-sleeper
  ```
  
## Entrypoint Instruction
- EntryPoint는 컨테이너가 시작될 때 실행될 프로그램 명령어와 명령 옵션을 지정하는 Commmand와 유사하다.
```
     command: ["sleep2.0"]
     args: ["10"]
     > docker run ubuntu-sleeper sleep 10
```
- 기존 위와 같던 command, arg에서 엔트리포인트를 사용하면 아래처럼 arg만 지정하여 사용할 수 있다.
```
     entrypoint: ["sleep"]
     > docker run ubuntu-sleeper 10
```

#### K8s Reference Docs
- https://docs.docker.com/engine/reference/builder/#cmd

# Commands and Arguments in Kubernetes
- docker run 명령에 추가되는 모든 항목은 배열 형식으로 포드 정의 파일의 **`args` ** 속성으로 이동합니다.
- POD를 생성할 때, POD 안에서 동작하는 컨테이너를 위한 command와 args를 정의할 수 있다.
- docker file의 Entrypoint에 해당하는 것은 **`command`** 필드이고, CMD에 해당하는 것은 **`args`** 이다.
- 정의한 command와 args는 POD가 생성되고 난 이후에는 변경될 수 없다
- 매니페스트 파일 안에서 정의하는 command와 args는 컨테이너 도커 이미지가 제공하는 기본 커맨드와 인자들보다 우선시 된다.
- 만약 args를 정의하고 command를 정의하지 않는다면, 도커 파일의 기본 CMD가 pod definition의 args와 함께 사용된다.

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu-sleeper-pod
  spec:
   containers:
   - name: ubuntu-sleeper
     image: ubuntu-sleeper
     command: ["sleep2.0"]
     args: ["10"]
  ```
  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/args.PNG>
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/05-Application-Lifecycle-Management/05-Commands-and-Arguments-in-Kubernetes.md
https://github.com/jonnung/cka-practice/blob/master/4-application-lifecycle-management.md
