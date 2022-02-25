# CONSUL 설치

## Environment
- [Ubuntu 20.04 LTS]
- kubeadm version: &version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:24:08Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/amd64"}

## Installation

### HashiCorp Helm Repository 추가
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/consul # Verify
```

### Namespace 가 없으면 생성
```
kubectl create ns consul
```

### 설치 전 필요 요소 생성
- 참고
  - files/config.yaml
  - files/pv.yaml

### pv 생성
```
k apply -f pv.yaml
```

### 설치
```
helm install consul hashicorp/consul --values config.yaml -n consul
```

### 삭제
```
helm uninstall consul -n consul
```

### 포트를 확인하고 ui에 접속
```
kubectl get svc -n consul

kubectl --namespace consul port-forward service/consul-consul-ui 18500:80 --address 0.0.0.0
```
- 18500 포트로 접속

## Error
- [ consul-server ] failed to setup node ID: failed to write NodeID to disk: open /consul/data/node-id: permission denied
  - https://githubhot.com/repo/hashicorp/consul-helm/issues/801?page=2
  ```
  server:
    replicas: 1
    bootstrapExpect: 1
    affinity: ""
    securityContext:
      fsGroup: 2000
      runAsGroup: 2000
      runAsNonRoot: false
      runAsUser: 0
  ```
- [ consul-client ] agent.anti_entropy: failed to sync remote state: error="No known Consul servers"
  - UDP 포트 확인


## 참고
https://www.consul.io/docs/k8s/connect/ingress-gateways
