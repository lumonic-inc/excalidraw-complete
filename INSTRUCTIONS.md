##### Setup Instructions 
This fork exists to override the URLs before it is compiled.


### Update apt package index
sudo apt update

### Install zsh
```
sudo apt install -y zsh && \
sudo apt-get install -y powerline fonts-powerline && \
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" && \
sudo chsh -s /bin/zsh ubuntu
nano ~/.zshrc 
export PROMPT="%F{magenta}[HOSTED]:$EC2_INSTANCE_ID%f $PROMPT"
# Set theme:  
ZSH_THEME="sonicradish"
```

### Install prerequisite packages
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### Add the repository to Apt sources:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Install required go version
```
wget https://go.dev/dl/go1.22.1.linux-arm64.tar.gz
sudo tar -C /usr/local -xzf go1.22.1.linux-arm64.tar.gz
```

### Add Go binary path to bashrc
```
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc
echo 'export PATH=$PATH:/usr/bin' >> ~/.zshrc
source ~/.zshrc
```

### Install nginx
```
sudo apt update
sudo apt install nginx
```
```
#Paste in the nginx config
nano /etc/nginx/conf.d/excalidraw.conf
# Paste the nginx-excalidraw.conf
sudo nano /etc/nginx/nginx.conf
# comment out enabled-sites
```

### Install the cert after nginx
```
sudo apt install certbot
sudo certbot --nginx -d draw.lumonic.com
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d draw.lumonic.com
```

### Make dir for logs
```
mkdir /var/www/excalidraw-complete
mkdir /var/www/excalidraw-complete/logs
```

### Restart nginx
```
sudo service nginx restart
```

### Setup supervisor
```
sudo apt install supervisor
# Copy in config
sudo nano /etc/supervisor/conf.d/excalidraw.conf
sudo supervisorctl reread
sudo supervisorctl update
```


### Add SSH key to authorized_keys
```
sudo nano ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa
```

### Configure SSH
```
echo "Host github
    Hostname github.com
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
    User ubuntu" >> ~/.ssh/config
```

### Change ownership of /var/www
```
sudo chown ubuntu:ubuntu /var/www
cd /var/www
```

### Install AWS CLI (For S3 Storage)
```
sudo apt-get update
sudo apt-get install awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
# Unzip and install
unzip awscliv2.zip
sudo ./aws/install

```

### Build UI
```
git clone git@github.com:lumonic-inc/excalidraw-complete.git --recursive
cd excalidraw-complete/excalidraw
git remote add jcobol https://github.com/jcobol/excalidraw
git fetch jcobol
git checkout 7582_fix_docker_build
git apply ../frontend.patch
cd ../
sudo docker build -t exalidraw-ui-build excalidraw -f ui-build.Dockerfile
sudo docker run -v ${PWD}/:/pwd/ -it exalidraw-ui-build cp -r /frontend /pwd
```

### Compile the go application
```
go build -o excalidraw-complete main.go
```

### Restart the application
```
sudo supervisorctl restart all
```
