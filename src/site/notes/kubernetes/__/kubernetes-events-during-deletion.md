---
{"author":"jx2lee","aliases":"pod 가 삭제되는 과정을 살펴봐요","created":"2023-12-20T00:33:04.000+09:00","last-updated":"2023-08-03 23:31","tags":["kubernetes","pod-deletion"],"dg-publish":true,"permalink":"/kubernetes/__/kubernetes-events-during-deletion/","dgPassFrontmatter":true,"noteIcon":""}
---


> **Pod 가 삭제 될 때 클러스터에서 어떤 일이 발생하는지 살펴본다.**

### Events
- pod 삭제 요청이 API 서버에 수신됨
- etcd 에서 상태를 수정 한 다음 삭제를 알림
	- 이러한 noti 를 알리는 주체들은 kubelet ,Endpoints Controller,kube-controller-manager
- 병렬로 발생하는 두 가지 이벤트(A 와 B로 표시)를 그림으로 보면 다음과 같음

![|700](https://i.imgur.com/y1Nav1w.png)

### A event
- etcd 상태 수정 후 API server 를 통해 kubelet 에게 pod 를 종료해야한다는 알림을 전송하며 종료 시퀀스를 시작함
	- *pre-stop-hook*
	- SIGTERM 전송
	- 컨테이너가 종료되지 않은 경우 강제 종료
- 앱이 클라이언트 요청 수신을 즉시 중단하고 SIGTERM에 응답하는 경우, 연결을 시도하는 모든 클라이언트는 연결 거부 오류를 수신함
- API 서버에서 kubelet 으로 파드가 삭제 된 시간부터 걸리는 시간은 비교적 짧음
- 정리하면,
	- `A1`: API server 가 해당 Pod 가 기동되는 Node 의 Kubelet 에게 종료 요청
	- `A2`: Kubelet 이 종료 시퀀스 시작
	    - A2-(1): pre-stop-hook
	    - A2-(2): SIGTERM 전송
	    - A2-(3): 종료되지 않은 경우 강제로 종료 수행

### B event
- 정리하면,
	- `B1`: API server 는 controller manager 에게 삭제 알림을 전달
	- `B2`: controller manager 는 삭제될 pod 가 속한 모든 서비스의 endpoints 삭제
	- `B3`: endpoint 수정을 위해 API server 는 각 node 의 kube-proxy 에게 수정요청
	- `B4`: 삭제될 Pod 가 속한 node 의 kube-proxy 는 대상 Pod 관련 iptables 삭제

### Parallel events
- 두 이벤트(`A`, `B`)는 모두 병렬로 수행함
- Pod 를 종료하는데 걸리는 시간은 iptables 규칙업데이트 보다 시간이 짤음
	- 이는 iptables 규칙을 업데이트 하는 체인이 길.
	- API 서버에 새 요청을 보내고 Endpoints 컨트롤러에 도달한 다음 API 서버가 또 이를 알려함
    - API server 를 여러 번 거쳐가야 함
- 모든 노드의 iptables 규칙이 업데이트 되기 전 SIGTERM 신호가 전송 될 가능성이 높음

![|700](https://i.imgur.com/RPDTsKf.png)

### reference

- [https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/](https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/)
