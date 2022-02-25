# RKE2

## RKE2 config file 이 위치할 폴더를 만들어줍니다.

```
mkdir -p /etc/rancher/rke2/
```

## 설치 커맨드 실행

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=v1.20 sh -

systemctl enable rke2-server.service

systemctl start rke2-server.service
```

## RKE2가 실행중인지 확인

```
# 준비상태인 노드를 확인할 수 있음
/var/lib/rancher/rke2/bin/kubectl \
        --kubeconfig /etc/rancher/rke2/rke2.yaml get nodes

# 준비상태인 파드를 확인할 수 있음
/var/lib/rancher/rke2/bin/kubectl \
        --kubeconfig /etc/rancher/rke2/rke2.yaml get pods --all-namespaces
```

## kubeconfig 파일을 사용하여 시작

1. `kubectl` 설치
2. `/etc/rancher/rke2/rke2.yaml` 을 `~/.kube/config` 로 복사
3. kubectl 커맨드 사용 가능하도록 설정
        ```
        exit
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```

자세한 내용은 아래 링크를 참고
- [kubectl 설치] https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- https://rancher.com/docs/rancher/v2.6/en/installation/resources/k8s-tutorials/ha-rke2/#3-save-and-start-using-the-kubeconfig-file

## 참고
- https://rancher.com/docs/rancher/v2.6/en/installation/resources/k8s-tutorials/ha-rke2