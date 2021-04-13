## ETCD
     - 간단하고 안전하며 빠른 분산 된 신뢰할 수있는 키-값 저장소
       
## Install ETCD
- github 릴리스 페이지에서 운영 체제에 대한 관련 바이너리를 다운로드(https://github.com/etcd-io/etcd/releases
     ```
       ETCD V3.3.11 curl command
       
       $ https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz
     ```
     - 압축 풀기.
       ```
       $ tar xvzf etcd-v3.3.11-linux-amd64.tar.gz 
       ```
     - ETCD 서비스 실행
       ```
       $ ./etcd
       ```
     - **`ETCD`** 실행하면 default로 포트 **`2379`** Listen
     - **`ETCD`** 와 함께 제공되는 기본 클라이언트는 **`etcdct`** 클라이언트, 이를 사용하여 Key-Value 쌍 저장, 검색 가능
        ```
        Syntax: Key-Value 페어 저장
        $ ./etcdctl set key1 value1
        ```
        ```
        Syntax: 저장된 데이터 검색
        $ ./etcdctl get key1
        ```
        ```
        Syntax: help 명령어
        $ ./etcdctl
        ```
        
        <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/etcdctl.PNG>
        
       K8s Reference Docs:
       - https://kubernetes.io/docs/concepts/overview/components/
       - https://etcd.io/docs/
       - https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
       
