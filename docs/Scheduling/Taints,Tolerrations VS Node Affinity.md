# Taints and Tolerations vs Node Affinity
- Taints and Tolerations -> 특정 노드에 스케쥴링되지 못하도록 막는 것 ==> 다른걸 막더라도 내가 원하는게 다른곳으로 갈 수있음
- Node Affinity -> 특정 Pod가 특정 Node에 스케쥴링되도록 하는 것 ==> 다른 Pod도 Node로 올 수 있음
- Taint와 Toleration은 POD가 어떤 노드만 선호한다고 보장하지는 않는다.
- Taint와Toleration 그리고 NodeAffinity를 함께 사용하면 특정 POD만 스케줄링되는 전용 노드를 만들 수 있다.

- 동일한 노드에 여러 Taint를 지정하거나 동일한 POD에 여러 Toleration을 지정할 수 있다.
  쿠버네티스가 여러 Taint 및 Toleration을 처리하는 방식은 필터와 같다.
  모든 노드의 Taint로 시작한 다음, POD에 일치하는 Toleration이 있는 것만 무시한다. 무시되지 않은 나머지 Taint는 POD에 표시된 effect를 적용한다.

  
#### K8s Reference Docs:
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/

참고 : https://github.com/jonnung/cka-practice/blob/master/2-scheduling.md
