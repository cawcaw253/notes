# single service 생성 예시

## static-server 생성
example/static-server.yaml 참고

## ingress-gateway 생성
example/ingress-gateway.yaml 참고

## intentions 작성
example/service-intentions.yaml 생성
혹은
consul ui 로부터 intentions 생성

## 접속 테스트
아래의 install.sh 예시를 참고 하였다면
서비스 consul-ingress-gateway 가 NodePort 30080 -> 8080와 같이 설정되어 있을 것이다.

따라서 브라우저에서 [Cluster IP]:30080 으로 접속하면 "hello world" 가 표시된 것을 볼 수 있다.


## 참고
https://www.consul.io/docs/k8s/connect/ingress-gateways
https://discuss.hashicorp.com/t/minikube-consul-ingress-gateway/29684