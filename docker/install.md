# Docker

## Environment
- [Ubuntu 20.04 LTS]

## Installation
1. Set up repository
    ```
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
2. Add Docker GPG key
    - https://docs.docker.com/engine/install/ubuntu/
    ```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
3. Set up stable repository
    ```
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
4. Install
    ```
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io
    ```

## Error
- permission denied
  ```
  sudo groupadd docker
  sudo usermod -aG docker $USER
  newgrp docker
  ```