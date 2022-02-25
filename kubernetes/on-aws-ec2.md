# Create Kubernetes Cluster with EC2 and Attach LoadBalancer

## Create VPC

### 예시
- IPv4 CIDR : 10.0.0.0/16
- Tags
  - Name : example-vpc
  - kubernetes.io/cluster/kubernetes : owned

### 생성 후
- `DNS 호스트 이름 편집` > `DNS 호스트 이름` 활성화

## Create Subnet

### 예시
- IPv4 CIDR: 10.0.0.0/16
- Tags
  - Name : example-subnet
  - kubernetes.io/cluster/kubernetes : owned

### 생성 후
- `서브넷 설정 편집` > `자동 할당 IP 설정` > `퍼블릭 IPv4 주소 자동 할당 활성화` 활성화

## Create Internet Gateway

### 예시
- Tags
  - Name : example-ig
  - kubernetes.io/cluster/kubernetes : owned

### 생성 후
- vpc(`example-vpc`)에 연결

## Create Routing Table

### 예시
- VPC : vpc(`example-vpc`)
- Tags
  - Name : example-rt
  - kubernetes.io/cluster/kubernetes : owned

### 생성 후
- `라우팅 편집` > 대상 추가
  - `0.0.0.0/0` > internet_gateway(`example-ig`)
- `서브넷 연결 편집` > subnet(`example-subnet`) 연결 저장

## Create IAM role and Policy

### IAM Master role for ec2
- Name : example-master-role
- Policy
  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "autoscaling:DescribeAutoScalingGroups",
                  "autoscaling:DescribeLaunchConfigurations",
                  "autoscaling:DescribeTags",
                  "ec2:DescribeInstances",
                  "ec2:DescribeRegions",
                  "ec2:DescribeRouteTables",
                  "ec2:DescribeSecurityGroups",
                  "ec2:DescribeSubnets",
                  "ec2:DescribeVolumes",
                  "ec2:CreateSecurityGroup",
                  "ec2:CreateTags",
                  "ec2:CreateVolume",
                  "ec2:ModifyInstanceAttribute",
                  "ec2:ModifyVolume",
                  "ec2:AttachVolume",
                  "ec2:AuthorizeSecurityGroupIngress",
                  "ec2:CreateRoute",
                  "ec2:DeleteRoute",
                  "ec2:DeleteSecurityGroup",
                  "ec2:DeleteVolume",
                  "ec2:DetachVolume",
                  "ec2:RevokeSecurityGroupIngress",
                  "ec2:DescribeVpcs",
                  "elasticloadbalancing:AddTags",
                  "elasticloadbalancing:AttachLoadBalancerToSubnets",
                  "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                  "elasticloadbalancing:CreateLoadBalancer",
                  "elasticloadbalancing:CreateLoadBalancerPolicy",
                  "elasticloadbalancing:CreateLoadBalancerListeners",
                  "elasticloadbalancing:ConfigureHealthCheck",
                  "elasticloadbalancing:DeleteLoadBalancer",
                  "elasticloadbalancing:DeleteLoadBalancerListeners",
                  "elasticloadbalancing:DescribeLoadBalancers",
                  "elasticloadbalancing:DescribeLoadBalancerAttributes",
                  "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                  "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                  "elasticloadbalancing:ModifyLoadBalancerAttributes",
                  "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                  "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                  "elasticloadbalancing:AddTags",
                  "elasticloadbalancing:CreateListener",
                  "elasticloadbalancing:CreateTargetGroup",
                  "elasticloadbalancing:DeleteListener",
                  "elasticloadbalancing:DeleteTargetGroup",
                  "elasticloadbalancing:DescribeListeners",
                  "elasticloadbalancing:DescribeLoadBalancerPolicies",
                  "elasticloadbalancing:DescribeTargetGroups",
                  "elasticloadbalancing:DescribeTargetHealth",
                  "elasticloadbalancing:ModifyListener",
                  "elasticloadbalancing:ModifyTargetGroup",
                  "elasticloadbalancing:RegisterTargets",
                  "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                  "iam:CreateServiceLinkedRole",
                  "kms:DescribeKey"
              ],
              "Resource": [
                  "*"
              ]
          }
      ]
  }
  ```

### IAM Worker role for ec2
- Name : example-worker-role
- Policy
  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "ec2:DescribeInstances",
                  "ec2:DescribeRegions",
                  "ecr:GetAuthorizationToken",
                  "ecr:BatchCheckLayerAvailability",
                  "ecr:GetDownloadUrlForLayer",
                  "ecr:GetRepositoryPolicy",
                  "ecr:DescribeRepositories",
                  "ecr:ListImages",
                  "ecr:BatchGetImage"
              ],
              "Resource": "*"
          }
      ]
  }
  ```

## Create Security Group

- VPC : vpc(`example-vpc`)
- Inbound rules
  - `All traffic`
- Tags
  - Name : example-sg
  - kubernetes.io/cluster/kubernetes : owned

## Create EC2

- Master
  - AMI : [Ubuntu Server 18.04 LTS (HVM)]
  - Configure Instance Details
    - Network
      - vpc(`example-vpc`)
      - subnet(`example-subnet`)
    - IAM role : role(`example-master-role`)
  - Tags
    - Name : example-master
    - kubernetes.io/cluster/kubernetes : owned
  - Security Group : security_group(`example-sg`)
  - AMI : [Ubuntu Server 18.04 LTS (HVM)]
- Worker
  - Configure Instance Details
    - Network
      - vpc(`example-vpc`)
      - subnet(`example-subnet`)
    - IAM role : role(`example-worker-role`)
  - Tags
    - Name : example-worker
    - kubernetes.io/cluster/kubernetes : owned
  - Security Group : security_group(`example-sg`)

## Setting Preferences

모든 노드에서 아래의 명령을 실행
(Ubuntu Server 18.04 LTS (HVM) 기준)

- Docker 설치
  1. 패키지 업데이트
      ```
      sudo apt update && apt -y upgrade
      ```
  2.  Docker 레파지토리 추가
      ```
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
      ```
  3. 설치
      ```
      sudo apt update
      sudo apt install docker-ce
      ```

  - [Error] permission denied 대응
      ```
      sudo groupadd docker
      sudo usermod -aG docker $USER
      newgrp docker
      ```

- Kubernetes 설치
  1. gpg key 추가
      ```
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
      ```
  2. 레파지토리 추가
      ```
      sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
      ```
  3. kubelet kubeadm kubectl 설치
      ```
      sudo apt update
      sudo apt install -y kubelet kubeadm kubectl
      ```

- Hostname 수정
  1. FDQN(Fully Qualified Domain Name) 취득
      ```
      curl http://169.254.169.254/latest/meta-data/local-hostname
      # (ex) ip-10-0-0-102.eu-west-3.compute.internal
      ```
  2. Hostname을 FDQN 값으로 설정
      ```
      sudo hostnamectl set-hostname ip-10-0-0-102.eu-west-3.compute.internal
      ```
  3. 확인
      ```
      hostname
      # ip-10-0-0-102.eu-west-3.compute.internal
      ```
- 기타
  - Docker cgroup driver 대응
    ```
    # vi /etc/docker/daemon.json
    {
      "exec-opts": ["native.cgroupdriver=systemd"]
    }

    sudo systemctl daemon-reload
    sudo systemctl restart docker
    sudo systemctl restart kubelet
    ```

## Init Cluster
- Master Node
  - config.yaml 파일 작성
    ```
    # vi /etc/kubernetes/aws.yaml

    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    networking:
      serviceSubnet: "10.100.0.0/16"
      podSubnet: "10.244.0.0/16"
    apiServer:
      extraArgs:
        cloud-provider: "aws"
    controllerManager:
      extraArgs:
        cloud-provider: "aws"
    ```
  - Init
    ```
    # root 유저로 실행
    kubeadm init --config /etc/kubernetes/aws.yml
    ```
  - kubelet config 파일 작성
    ```
    # 일반 유저로 실행
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
- Worker Node
  - Master Init 시에 나왔던 Join 커맨드 입력
- AWS
  - 클러스터에서 로드밸런서 생성 후 aws 콘솔에 가면 로드밸런서가 생성된 것을 알 수 있음
  - `생성된 로드 밸런서` > `인스턴스` > `인스턴스 편집` 에서 워커노드를 추가

## ERROR

### Core DNS 와 CNI 가 먹통일 때
  1. /etc/kubernetes/manifests/kube-controller-manager.yaml 을 수정
      ```
      # command 부분에 아래의 내용을 추가 (flannel cni 기준)
      --allocate-node-cidrs=true
      --cluster-cidr=10.244.0.0/16
      ```
  2. kubelet을 재실행
  - 참고 : https://github.com/flannel-io/flannel/issues/728#issuecomment-325347810

## 참고

- https://rtfm.co.ua/en/kubernetes-part-2-a-cluster-set-up-on-aws-with-aws-cloud-provider-and-aws-loadbalancer/
