# OS Upgrades

#### 노드에 문제가 생겨서 해당 노드에 실행 중인 POD는 5분 안에(kube-controller-manger의 --pod-eviction-timeout 옵션에 따름 - default 5 min) 정상화 되지 않을 경우 다른 노드에 스케줄링 된다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/os.PNG>
  
- 모든 작업 부하의 노드를 의도적으로 **`drain`**하여 작업 부하가 다른 노드로 이동되도록 할 수 있습니다.
- drain 명령은 gracefully 하게 POD를 종료시키고 다른 노드에 POD를 재생성하게 한다. 그리고 해당 노드는 스케줄링 되지 않게 마킹된다. (cordoned 상태)
  ```
  $ kubectl drain node-1
  ```
- 노드도 연결되거나 예약 불가능으로 표시됩니다.
- 유지 관리 후 노드가 다시 온라인 상태가 되더라도 여전히 예약 할 수 없습니다. 그런 경우 drain을 해제하고 난 후에 사용할 수 잇습니다.
  ```
  $ kubectl uncordon node-1
  ```
- cordon이라는 또 다른 명령도 있습니다. Cordon은 단순히 노드를 예약 불가능으로 표시합니다. 드레인과 달리 기존 노드에서 포드를 종료하거나 이동하지 않습니다.

  <img src = https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/images/drain.PNG>

- 만약 어떤 노드에 ReplicaSets의 일부가 아닌 단독으로 실행되는 POD가 있는데 해당 노드를 그냥 Drain하게 되면 그 POD는 자동으로 Evicted 되지 않는다.
그래서 그 POD를 강제로 제거하기 위해 --force 옵션을 사용할 수 있다. 하지만 그러면 그 POD는 다른 노드에 스케줄링 되지 않고 영원히 사라지게 된다.
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

참고 : https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/06-Cluster-Maintenance/02-OS-Upgrades.md
https://github.com/jonnung/cka-practice/blob/master/5-cluster-maintenance.md
