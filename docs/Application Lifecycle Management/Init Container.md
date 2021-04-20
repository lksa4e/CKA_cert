## InitContainers
POD의 컨테이너가 실행되기 전에 어떤 선행 과정을 수행해야 할 때가 있다.  
예를 들어 원격 저장소로부터 소스코드나 바이너리 파일을 내려받아 메인 애플리케이션에서 사용할 수 있도록 한다거나, POD가 생성될 때 반드시 1번은 실행되어야 하는 작업이 있을 수 있다.  
이러한 작업을 위해 사용하는 것이 InitContainers 이다.  

POD에 InitContainers를 설정하는 방법은 일반적인 컨테이너 선언하는 방식과 유사하지만 `initContainers` 섹션이 따로 존재한다.  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

POD가 처음 생성될 때 initContainers가 먼저 실행되며, initContainers 실행이 완전히 종료되어야 실제 컨테이너가 실행될 수 있다.  
initContainers도 여러개 설정할 수 있다. 이 때는 InitContainers를 선언한 순서대로 1번씩 실행된다.  
만약 initContainers 중에 어떤 한 개라도 실패하면 쿠버네티스는 해당 POD를 재실행하여 InitContainers 수행이 완전히 성공할 때까지 반복하게 된다.  

참고 : https://github.com/jonnung/cka-practice/blob/master/4-application-lifecycle-management.md
