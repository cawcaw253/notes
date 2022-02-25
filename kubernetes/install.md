# [Ubuntu 20.04 LTS]

## Install kubeadm, kubectl, kubelet

```
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00

sudo apt-mark hold kubelet kubeadm kubectl
```

## Set user as root

```
sudo -i
```

## Turn off swap 

```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Change docker cgroup driver

```
# vi /etc/docker/daemon.json

{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
```

```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

## ----- Master -----

```
kubeadm init
```

### CNI 설치 (weave net)

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

## ----- Worker -----

```
kubeadm join 10.0.1.229:6443 --token bcarry.goqxygfuoeygum90 \
        --discovery-token-ca-cert-hash sha256:43143f4937dfe259b7610e4bc27f15c410d7111578f85b718c5bf15e9b423437

```

### kubectl 커맨드 사용 가능하도록 설정

```
exit
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join 커맨드 출력

```
kubeadm token create --print-join-command
```

## Check

```
docker info | grep -i Cgroup    # docker cgroup
systemctl status kubelet        # kubelet 상태 확인, CNI 가 설치되기 전까지는 loaded 상태가 표시됨 (kubeadm init 혹은 join 을 실행해야 함)
journalctl -u kubelet
kubectl get pod -A
```