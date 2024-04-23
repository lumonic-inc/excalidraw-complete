#!/bin/bash

```
# Update apt package index
sudo apt update

# Install prerequisite packages
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository to APT sources
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

# Update package database with Docker packages
sudo apt update

# Check Docker version candidate
apt-cache policy docker-ce

# Install Docker
sudo apt install -y docker-ce

# Install required go version
sudo apt-get remove --purge golang-go
wget https://go.dev/dl/go1.22.1.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz

# Add Go binary path to bashrc
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export PATH=$PATH:/usr/bin' >> ~/.bashrc
source ~/.bashrc

# Prompt user to paste SSH key
echo "Please paste your SSH key below and press Enter:"
read -r ssh_key

# Add SSH key to authorized_keys
mkdir -p ~/.ssh && echo "$ssh_key" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa

# Configure SSH
echo "Host github
    Hostname github.com
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    User ubuntu" >> ~/.ssh/config

# Change ownership of /var/www
sudo chown ubuntu:ubuntu /var/www
cd /var/www

# Build UI
git clone git@github.com:lumonic-inc/excalidraw-complete.git --recursive
cd excalidraw-complete/excalidraw
git remote add jcobol https://github.com/jcobol/excalidraw
git fetch jcobol
git checkout 7582_fix_docker_build
git apply ../frontend.patch
cd ../
sudo docker build -t exalidraw-ui-build excalidraw -f ui-build.Dockerfile
docker run -v ${PWD}/:/pwd/ -it exalidraw-ui-build cp -r /frontend /pwd

# Compile the go application
go build -o excalidraw-complete main.go
```