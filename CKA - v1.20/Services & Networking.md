# Understand host networking configuration on the cluster nodes

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/networking.png>

호스트 레벨에는 기본 네트워크 어댑터 역할을하는 인터페이스 (일반적으로`eth0` 또는`ens192` 등)가 있습니다.  

각 호스트는 CNI 범위의 서브넷 하나를 담당합니다. 이 예에서 왼쪽 호스트는 10.1.1.0/24를 담당하고 오른쪽 호스트는 10.1.2.0/24를 담당합니다. 전체 포드 CIDR 블록은 10.1.0.0/16과 비슷할 수 있습니다.

가상 이더넷 어댑터는 해당 포드 네트워크 어댑터와 페어링됩니다. 커널 라우팅은 Pod가 상주하는 호스트 외부에서 통신 할 수 있도록하는 데 사용됩니다.

# Understand connectivity between Pods

모든 포드는 자체 IP 주소를 갖습니다. 즉, Pod간에 명시 적으로 링크를 만들 필요가 없으며 컨테이너 포트를 호스트 포트에 매핑하는 작업을 거의 처리 할 필요가 없습니다. 
이렇게하면 포트 할당, 이름 지정, 서비스 검색, 부하 분산, 애플리케이션 구성, 마이그레이션의 관점에서 Pod가 VM 또는 물리적 호스트와 매우 유사하게 처리 될 수있는 깨끗한 이전 버전과 호환되는 모델이 생성됩니다.

Kubernetes는 모든 네트워킹 구현에 다음과 같은 기본 요구 사항을 적용합니다 (의도적 인 네트워크 세분화 정책 제외):

*   노드의 포드는 NAT없이 모든 노드의 모든 포드와 통신 할 수 있습니다
*   노드의 에이전트 (예 : 시스템 데몬, kubelet)는 해당 노드의 모든 포드와 통신 할 수 있습니다.

Note: `hostNetwork`를 활용하는 워크로드를 실행할 때:

*   노드의 호스트 네트워크에있는 포드는 NAT없이 모든 노드의 모든 포드와 통신 할 수 있습니다

# Understand ClusterIP, NodePort, LoadBalancer service types and endpoint

Pods는 임시적입니다. 따라서 안정적이고 정적인 진입점을 제공하는 Service 뒤에 Pod를 배치하는 것이 kubernetes Service 객체의 기본 사용법입니다. 
서비스는 다음과 같은 형식을 취합니다:

*   ClusterIP - Internal only
*   LoadBalancer - External, requires cloud provider, or software implementation to provide one
*   NodePort - External, 노드에 직접 액세스 해야함
*   Ingress resource - L7 Ingress는 서비스를 외부에서 연결할 수있는 URL, 부하 분산 트래픽, SSL 종료, 이름 기반 가상 호스팅 제공하도록 구성 할 수 있습니다. 
인그레스 컨트롤러는 일반적으로 로드 밸런서를 사용하여 인 그레스를 수행 할 책임이 있지만 트래픽 처리를 돕기 위해 에지 라우터 또는 추가 프런트 엔드를 구성 할 수도 있습니다.


# Know how to use Ingress controllers and Ingress resources

Ingress는 클러스터 외부에서 클러스터 내의 서비스로 HTTP 및 HTTPS 경로를 노출합니다. Ingress는 두 가지 구성 요소로 구성됩니다. 
Ingress Resource는 인바운드 트래픽이 서비스에 도달하기위한 규칙 모음입니다. 이는 호스트 이름 (및 선택적으로 경로)이 Kubernetes의 특정 서비스로 향하도록 허용하는 7계층(L7) 규칙입니다. 
두 번째 구성 요소는 일반적으로 HTTP 또는 L7 로드 밸런서를 통해 Ingress Resource에서 설정 한 규칙에 따라 작동하는 인그레스 컨트롤러입니다. 
외부 클라이언트에서 Kubernetes 서비스로 트래픽을 라우팅하도록 두 부분을 모두 적절하게 구성하는 것이 중요합니다..

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/ingress.png>

다음 yaml은 foo.bar.com 웹 사이트에 대한 두 개의 수신 규칙을 생성합니다.

기본 경로는 포트 80에서 수신하는 서비스 "default-service"로 트래픽을 보냅니다

/ foo로 끝나는 경로는 포트 4200에서 수신하는 서비스 "service1"로 트래픽을 보냅니다.

/ bar로 끝나는 경로는 포트 8080에서 수신하는 서비스 "service2"로 트래픽을 보냅니다.


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-fanout-example
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /
        backend:
          serviceName: default-service
          servicePort: 80
      - path: /foo
        backend:
          serviceName: service1
          servicePort: 4200
      - path: /bar
        backend:
          serviceName: service2
          servicePort: 8080
```

Ingress 리소스가 작동하려면 클러스터에 Ingress 컨트롤러가 실행 중이어야합니다. Ingress 컨트롤러는 Kubernetes 클러스터에 워크로드로 배치됩니다:

```shell
> kubectl get po -A | grep nginx-ingress
ingress-nginx              nginx-ingress-controller-2gxtd                            1/1     Running     0          14d
ingress-nginx              nginx-ingress-controller-9lrzh                            1/1     Running     0          14d
ingress-nginx              nginx-ingress-controller-r2ksq                            1/1     Running     0          14d
```

# Know how to configure and use CoreDNS

1.13 부터, coredns는 클러스터 DNS의 진행자로 kube-dns를 대체하고 포드로 실행됩니다.

```shell
kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-hxbhn                 1/1     Running   9          10d
coredns-fb8b8dccf-jks6g                 1/1     Running   4          8d
```

포드의 DNS 구성을 보려면 포드를 실행하고 /etc/resolv.conf를 검사하십시오.:


```shell
kubectl run busybox --image=busybox -- sleep 9000
kubectl exec -it busybox sh
/ # cat /etc/resolv.conf  
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local virtualthoughts.co.uk
options ndots:5
```

"10 .96.0.10" 은 `Kube-DNS` 서비스를 참조합니다.

“Default.svc.cluster.local” 는 접미사가 svc.cluster.local 인 네임스페이스를 참조합니다.

모든 포드는 DNS 레코드로 프로비저닝되며 다음 형식입니다.

**[Pod IP separated by dashes].[Namespace].[type].[Base Domain Name]**

이 예에서 `[type]`이`pod` 인 경우 put 서비스는 동일한 규칙을 통해 확인할 수 있습니다.

For example:

```shell
/ # nslookup 10-42-2-68.default.pod.cluster.local
Server:		10.43.0.10
Address:	10.43.0.10:53

Name:	10-42-2-68.default.pod.cluster.local
Address: 10.42.2.68

```

Services도 비슷한 패턴을 가집니다.

**[Service Name].[Namespace].[type].[Base Domain Name]**

For example:

```shell
my-svc.my-namespace.svc.cluster-domain.example
```

헤드리스 서비스는 클러스터 IP가 없는 서비스이지만, 특정 시점에 적용 가능한 IP 포드 목록으로 응답합니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-headless
```

Pod definition yaml 파일에서 포드 dns 구성의 기본 동작을 수정할 수 있습니다:

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

CoreDNS에는 수정할 수있는 configmap도 있습니다:

```shell
kubectl get cm coredns -n kube-system -o yaml                                                            
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        hosts /etc/coredns/NodeHosts {
          reload 1s
          fallthrough
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
  NodeHosts: |
    172.16.10.100 k3s-ranch-node-1
    172.16.10.101 k3s-ranch-node-2
    172.16.10.102 k3s-ranch-node-3

```
Corefile 구성에는 다음과 같은 CoreDNS 플러그인이 포함됩니다:

* `errors`: 에러는 stdout에 로그로 기록됩니다.
* `health`: CoreDNS의 상태는 http://localhost:8080/health 에 보고됩니다. 이 확장 구문에서 lameduck은 프로세스를 비정상 상태로 만들고 프로세스가 종료되기 전에 5 초 동안 기다립니다.
* `ready`: 모든 플러그인이 준비상태가 가능한 경우, 포트 8181의 HTTP Endpoint는 200 OK를 반환합니다.
* `kubernetes`: CoreDNS는 Kubernetes의 서비스 및 포드의 IP를 기반으로 DNS 쿼리에 응답합니다. CoreDNS 웹 사이트에서 해당 플러그인에 대한 자세한 내용을 찾을 수 있습니다. 
TTL을 사용하면 응답에 대한 사용자 지정 TTL을 설정할 수 있습니다. 기본값은 5 초입니다. 허용되는 최소 TTL은 0 초이고 최대 값은 3600 초로 제한됩니다. 
TTL을 0으로 설정하면 레코드가 캐시되지 않습니다. pods insecure 옵션은 kube-dns와의 역 호환성을 위해 제공됩니다. 
IP가 일치하는 동일한 네임 스페이스에 포드가있는 경우에만 A 레코드를 반환하는 포드 확인 옵션을 사용할 수 있습니다. 
포드 레코드를 사용하지 않는 경우 포드 비활성화 옵션을 사용할 수 있습니다.
* `prometheus`: CoreDNS의 메트릭은 http://localhost:9153/metrics 에서 Prometheus 형식 (OpenMetrics라고도 함)으로 제공됩니다.
* `forward`: AKubernetes의 클러스터 도메인 내에 없는 쿼리는 사전정의된 Resolver(/etc/resolv.conf)로 전달됩니다. 
* `cache`: 프런트 엔드 캐시를 활성화합니다.
* `loop`: 단순 전달 루프를 감지하고, 루프가 발견되면 CoreDNS 프로세스를 중지합니다.
* `reload`: 변경된 Corefile을 자동으로 다시로드 할 수 있습니다. ConfigMap 구성을 편집 한 후 변경 사항이 적용될 때까지 2 분 정도 기다리십시오.

ConfigMap을 수정하여 기본 CoreDNS 동작을 수정할 수 있습니다.

# Choose an appropriate container network interface plugin

Pod가 서로 통신 할 수 있도록 CNI(Container Network Interface) 기반 Pod Network Add-on을 배포해야합니다. 네트워크가 설치되기 전에는 클러스터 DNS(CoreDNS)가 시작되지 않습니다

[https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy)

일반적으로 CNI는 일종의 네트워크 오버레이를 제공합니다. 그러나 각각 고유 한 기능, 제한 및 고려 사항이 있습니다.

CNI's manifest 및 Kubernetes pods는 클러스터 안에 있습니다. 일반적인 워크플로에는 k8s 클러스터를 구축하고 다음을 포함하는 네트워크 cni를 적용하는 작업이 포함됩니다:

```shell
kubectl apply -f <add-on.yaml>
```
