# EC2 Setup: Install Docker, Configure Network, and Deploy Containers

## Prerequisites
- AWS EC2 Instance (Amazon Linux 2)
- AWS CLI configured
- Docker installed
- AWS ECR repository access

---

## Install `yum` and Docker on EC2 Machine
```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker  # Start Docker
sudo systemctl enable docker  # Enable Docker
sudo systemctl status docker  # Verify Docker status
sudo usermod -aG docker ec2-user  # Add user to Docker group

# Exit and login again to apply group changes
exit
ssh -i <your-key.pem> ec2-user@<your-ec2-instance-ip>

# Verify group assignment
groups
```

---

## Create a Bridge Network
```bash
docker network create --driver bridge --subnet 192.168.2.0/24 --gateway 192.168.2.1 my-bridgenetwork
```

---

## Pull MySQL Image from AWS ECR and Run Container
```bash
ecr_dbimage=141947217581.dkr.ecr.us-east-1.amazonaws.com/clo835-assignment1-db:mysql-latest

# Authenticate Docker with AWS ECR
aws ecr get-login-password --region us-east-1 | docker login -u AWS $ecr_dbimage --password-stdin

# Run MySQL container
docker run -d \
  -e MYSQL_ROOT_PASSWORD=pw \
  --network my-bridgenetwork \
  --name mysql-db \
  $ecr_dbimage
```

### Verify Container Details
```bash
docker inspect mysql-db
```

---

## Database Connection and Verification
```bash
# Access MySQL container
docker exec -it mysql-db /bin/bash

# Connect to MySQL and show databases
mysql -uroot -ppw -e "SHOW DATABASES;"
```

### Export Environment Variables
```bash
export DBHOST=192.168.2.2  # IP obtained from `docker inspect`
export DBPORT=3306
export DBUSER=root
export DATABASE=employees
export DBPWD=pw
```

---

## Pull Web App Image from AWS ECR and Deploy Containers
```bash
ecr_appimage=141947217581.dkr.ecr.us-east-1.amazonaws.com/clo835-assignment1-app:app-latest

# Authenticate Docker with AWS ECR
aws ecr get-login-password --region us-east-1 | docker login -u AWS $ecr_appimage --password-stdin

# Deploy multiple web app containers in different colors
docker run -d -p 8081:8080 \
  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER \
  -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=blue \
  --network my-bridgenetwork --name blue_app $ecr_appimage

docker run -d -p 8082:8080 \
  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER \
  -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=pink \
  --network my-bridgenetwork --name pink_app $ecr_appimage

docker run -d -p 8083:8080 \
  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER \
  -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=lime \
  --network my-bridgenetwork --name lime_app $ecr_appimage
```

---

## Verify Container Communication
```bash
# Access blue_app container
docker exec -it blue_app /bin/bash

# Install `ping` tool if not available
apt-get update
apt-get install iputils-ping -y

# Ping other containers
targets=(pink_app lime_app)
for target in "${targets[@]}"; do
  ping -c 4 $target
done
```
