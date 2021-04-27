# TroubleShooting
Agenda
1. Evaluate cluster and node logging
2. Understand how to monitor applications
3. Manage container stdout & stderr logs
4. Troubleshoot application failure
5. Troubleshoot cluster component failure
6. Troubleshoot networking

Evaluate cluster and node logging - https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/ <br>
Understand how to monitor applications - https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/ <br>
Manage container stdout & stderr logs - https://kubernetes.io/docs/concepts/cluster-administration/logging/ <br>
Troubleshoot application failure - https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/ , https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/ <br>
Troubleshoot cluster component failure - https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/ <br>
Troubleshoot networking - https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/ <br>

# Evaluate cluster and node logging

## Master Node(s)

### ETCD

일반적으로 대부분의 etcd 구현에는 클러스터 상태를 모니터링하는 데 도움이 될 수있는 etcdctl도 포함됩니다. 어디서 찾을 수 있는지 확실하지 않은 경우 다음을 실행하십시오:

`find / -name etcdctl`

이 도구를 활용하여 클러스터 상태 확인:

```bash
./etcdctl cluster-health

member 17f206fd866fdab2 is healthy: got healthy result from https://master-0.etcd.cfcr.internal:2379
```

실행 된 클러스터에는 마스터 노드가 하나만 있으므로 스크립트의 결과는 하나뿐입니다. 일반적으로 클러스터의 각 etcd 구성원에 대한 응답을받습니다.

또는 kubectl get componentstatuses 활용:

```bash
kubectl get componentstatuses

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-1               Healthy   {"health":"true"}    
etcd-0               Healthy   {"health":"true"} 
```

Etcd는 Pod로 실행될 수도 있습니다:

```shell
kubectl logs etcd-ubuntu -n kube-system
```


### Kube-apiserver

이는 Kubernetes 플랫폼이 설치된 환경에 따라 다릅니다. For systemd based systems:

```bash
journalctl -u kube-apiserver
```
Or

```bash
cat /var/log/kube-apiserver.log
```

Or Kube-API 서버가 정적 포드로 실행되는 경우:

```bash
kubectl logs kube-apiserver-k8s-master-03 -n kube-system
```


### Kube-Scheduler

For systemd-based systems

```bash
journalctl -u kube-scheduler
```

Or

```bash
cat /var/log/kube-scheduler.log
```

Or for instances where Kube-Scheduler is running as a static pod:

```bash
kubectl logs kube-scheduler-k8s-master-03 -n kube-system
```


### Kube-Controller-Manager

For systemd-based systems

```bash
journalctl -u kube-controller-manager
```

Or

```bash
cat /var/log/kube-controller-manager.log
```

Or for instances where Kube-controller manager is running as a static pod:

```bash
kubectl logs kube-controller-manager-k8s-master-03 -n kube-system
```


## Worker Node(s)

## CNI

분명히 이것은 작업중인 클러스터에 사용중인 CNI에 따라 다릅니다. 그러나 Flannel을 예로 들어:

```bash
journalctl -u flanneld
```

If running as a pod, however:

```shell
Kubectl logs --namespace kube-system <POD-ID> -c kube-flannel
kubectl logs --namespace kube-system weave-net-pwjkj -c weave
```


### Kube-Proxy

For systemd-based systems

```shell
journalctl -u kube-proxy
```

Or

```shell
cat /var/log/kube-proxy.log
```

Or for instances where Kube-proxy manager is running as a static pod:

```bash
kubectl logs kube-proxy -n kube-system
```

### Kubelet

```shell
journalctl -u kubelet
```

Or

```shell
cat /var/log/kubelet.log
```

### Container Runtime

CNI와 마찬가지로 이는 배포 된 컨테이너 런타임에 따라 다르지만 Docker를 예로 사용합니다:

For systemd-based systems:

```shell
journalctl -u docker.service
```

Or

```shell
cat /var/log/docker.log
```

Hint : systemd 기반 서비스 인 경우`etc / systemd / system`의 내용을 나열합니다 (containerd.service가 여기에 있을 수 있음).

## Cluster Logging

At a cluster level, `kubectl get events` provides a good overview.

#  Understand how to monitor applications

이 섹션은 배포 한 항목과 애플리케이션의 토폴로지에 따라 크게 달라 지므로 약간 개방적입니다. 
그러나 일반적으로 상호 연결된 여러 **마이크로 서비스** 로 실행되는 애플리케이션이 있으므로 이를 구성하는 기본 개체를 모니터링하여 애플리케이션을 모니터링합니다.:

* Pods
* Deployments 
* Services
* etc

## Manage container stdout & stderr logs

<img src = https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/images/logging.png>

Kubernetes는 컨테이너 stdout 및 stderr 스트림에서 생성 된 모든 출력을 처리하고 리디렉션합니다. 이러한 로그는 로그 저장 위치에 영향을 미치는 로깅 드라이버를 통해 전달됩니다.
Docker의 구현 방식마다 정확한 구현 방식(예: RHEL의 Docker flavor)은 다르지만 일반적으로 이러한 드라이버는 파일에 json 형식으로 씁니다:

```shell
root@ubuntu:~# docker info | grep "Logging Driver"
 Logging Driver: json-file
```

이러한 로그의 위치는 일반적으로 `/var/log/containers`이지만 조정할 수 있습니다. 또한 이러한 항목에는 심볼 링크가 포함되어 있습니다:

```shell
root@ubuntu:~# ls -la /var/log/containers/
total 44
drwxr-xr-x  2 root root   4096 Feb  8 19:18 .
drwxrwxr-x 11 root syslog 4096 Feb 12 00:00 ..
lrwxrwxrwx  1 root root    100 Feb  8 19:17 coredns-74ff55c5b-j4trd_kube-system_coredns-5d65324791ffcdf45d3552d875c6834f9a305c5be84b18745cb1657f784e5dd0.log -> /var/log/pods/kube-system_coredns-74ff55c5b-j4trd_4afbef57-5592-4edb-96af-9d17f595d160/coredns/0.log
lrwxrwxrwx  1 root root    100 Feb  8 19:17 coredns-74ff55c5b-wrgkr_kube-system_coredns-b2fcfa679e9725dbe601bc1a0f218121a9c44b91d7300bbb57039a4edd219991.log -> /var/log/pods/kube-system_coredns-74ff55c5b-wrgkr_b64ac6d5-654b-4194-b6a8-5f8aa4c3cbe2/coredns/0.log
lrwxrwxrwx  1 root root     81 Feb  8 19:16 etcd-ubuntu_kube-system_etcd-fcc5bc99932f380781776baa125b6f3be035e18fcec520afb827102e2afce1cd.log -> /var/log/pods/kube-system_etcd-ubuntu_f608198a8b73b3cf090bd15e2823df04/etcd/0.log
lrwxrwxrwx  1 root root    101 Feb  8 19:16 kube-apiserver-ubuntu_kube-system_kube-apiserver-a264bbd54b7f23c8d424b0b368a48fdd1c5dcecc72fca95a460c146b2b5d85f5.log -> /var/log/pods/kube-system_kube-apiserver-ubuntu_212641053a16fa2bb404ccde20f6eaf0/kube-apiserver/0.log
lrwxrwxrwx  1 root root    119 Feb  8 19:17 kube-controller-manager-ubuntu_kube-system_kube-controller-manager-a4ef7fe2b52272ea77f8de2da0989a9bcee757ae778fc08f1786b26b45bf13e1.log -> /var/log/pods/kube-system_kube-controller-manager-ubuntu_7bbe7d37f1b2c7586237165580c2f5c3/kube-controller-manager/0.log
lrwxrwxrwx  1 root root    102 Feb  8 19:05 kube-flannel-ds-rfsfs_kube-system_install-cni-f8762e22fbf17925432682bdb1259a066208c62fa695d09cd6ee9b0cef3d36ba.log -> /var/log/pods/kube-system_kube-flannel-ds-rfsfs_2892d4e3-e326-4b4b-90c0-396fb80863ca/install-cni/0.log
lrwxrwxrwx  1 root root    103 Feb  8 19:05 kube-flannel-ds-rfsfs_kube-system_kube-flannel-f96e019717814d7360e1aacd275cac121c13e0ee94cc5c93dcb35365608e6f83.log -> /var/log/pods/kube-system_kube-flannel-ds-rfsfs_2892d4e3-e326-4b4b-90c0-396fb80863ca/kube-flannel/0.log
lrwxrwxrwx  1 root root     96 Feb  8 19:17 kube-proxy-l52f9_kube-system_kube-proxy-bfe08cb8663b46551e8608c094194ec61d03edfa7d25a6f414c07ed6563ada89.log -> /var/log/pods/kube-system_kube-proxy-l52f9_d2b73ed1-5df4-4a18-9595-20798db4f110/kube-proxy/0.log
lrwxrwxrwx  1 root root    101 Feb  8 19:17 kube-scheduler-ubuntu_kube-system_kube-scheduler-4a695e53684f4591ec9385d6944f7841c0329aa49be220e5af6304da281cb41a.log -> /var/log/pods/kube-system_kube-scheduler-ubuntu_69cd289b4ed80ced4f95a59ff60fa102/kube-scheduler/0.log
```

# Troubleshoot application failure

애플리케이션에 로그가 포함되어 있는 경우, 애플리케이션 장애 해결에 대한 접근 방식은 애플리케이션 아키텍처에 따라 활용 중인 리소스/API 개체에 따라 다르기 때문에 이 문제를 다루는 것은 어렵습니다.
그러나 좋은 출발점에는 다음과 같은 실행 방법이 포함될 수 있습니다:

* `kubectl describe <object>`
* `kubectl logs <podname>`
* `kubectl get events`

# Troubleshoot cluster component failure

Covered in "Evaluate cluster and node logging"

# Troubleshoot networking

## DNS Resolution

`Pods` 및`Services`는 클러스터의`coredns`에 대해 자동으로 등록 된 DNS 레코드 (IPv4의 경우 "A"레코드, IPv6의 경우 "AAAA")를 갖습니다. 
Format:

`pod-ip-address.my-namespace.pod.cluster-domain.example`
`my-svc-name.my-namespace.svc.cluster-domain.example`

Pod DNS 레코드는 동일한 네트워킹 공간을 공유하므로 Pod에 여러 컨테이너가 포함되어 있어도 단일 항목으로 확인됩니다.

서비스 DNS 레코드는 해당 서비스 개체로 확인됩니다.

포드는 coredns 설정에 따라 자동으로 DNS 확인을 구성합니다. 포드에 대한 셸을 열고 /etc/resolv.conf를 검사하여 확인할 수 있습니다:

```shell
> kubectl exec -it web-server sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
/ # cat /etc/resolv.conf 
nameserver 10.43.0.10
search default.svc.cluster.local svc.cluster.local cluster.local eu-central-1.compute.internal
options ndots:5
```

`10.43.0.10` 은 coredns service 객체입니다:

```shell
> kubectl get svc -n kube-system 
NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
kube-dns                     ClusterIP   10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP         16d
```

To test resolution, we can run a pod with `nslookup` to test. For the pod below:

```shell
> kubectl get po -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
web-server   1/1     Running   0          2d20h   10.42.1.31   ip-172-31-36-67   <none>           <none>
```

A 레코드의 형식 알기:

`pod-ip-address.my-namespace.pod.cluster-domain.example`

Tip : 클러스터 도메인을 확인하려면 coredns configmap을 검사하십시오. 아래는`cluster.local`을 나타냅니다..

```shell
> kubectl get cm coredns -n kube-system -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
```

필요한 도구로 Pod 만들기:

```shell
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

Test lookup:

```shell
kubectl exec -i -t dnsutils -- nslookup 10-42-1-31.default.pod.cluster.local
```

```shell
> kubectl exec -i -t dnsutils -- nslookup 10-42-1-31.default.pod.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10#53

Name:   10-42-1-31.default.pod.cluster.local
Address: 10.42.1.31
```

비슷하게, 이 케이스에서는 default namespace에 `nginx-service`라는 서비스가 있습니다:

```shell
> kubectl exec -i -t dnsutils -- nslookup nginx-service.default.svc.cluster.local
Server:         10.43.0.10
Address:        10.43.0.10#53

Name:   nginx-service.default.svc.cluster.local
Address: 10.43.0.223
```

```shell
> kubectl get svc
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.43.0.223   <none>        80/TCP    9m15s
```

## CNI Issues

발생할 수있는 한 가지 문제는 CNI가 잘못되었거나 초기화되지 않은 경우입니다. 이로 인해 워크로드가 '보류 중'상태가 될 수 있습니다:

```shell
kubectl get po -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx   0/1     Pending   0          57s   <none>   <none>   <none>           <none>
```

`kubectl describe <pod>`는 CNI에서 노드에 IP 주소를 할당하는 문제를 식별하는 데 도움이 될 수 있습니다

## Port Checking

마찬가지로, 'nslookup'을 활용하여 클러스터에서 DNS 확인을 검증하면 다른 도구를 사용하여 다른 진단을 수행 할 수 있습니다.
`netcat`,`telnet` 등과 같은 유틸리티가있는 포드 만 있으면됩니다.

참고 : https://github.com/David-VTUK/CKA-StudyGuide/blob/master/RevisionTopics/05-Troubleshooting.md
