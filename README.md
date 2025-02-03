Install yum and Docker into EC2 machine

sudo yum update -y
sudo yum install -y docker
sudo systemctl start docker // start
sudo systemctl enable docker // enable 
sudo systemctl status docker 
sudo usermod -aG docker ec2-user // 

exit and login again into the machine
groups
====
Bridge Network 
docker network create --driver bridge --subnet 192.168.2.0/24 --gateway 192.168.2.1 my-bridgenetwork

Pull ECR images from mysql repo

ecr_dbimage=141947217581.dkr.ecr.us-east-1.amazonaws.com/clo835-assignment1-db:mysql-latest


Login and Password
aws ecr get-login-password --region us-east-1 | docker login -u AWS $ecr_dbimage --password-stdin

docker run -d -e MYSQL_ROOT_PASSWORD=pw --network my-bridgenetwork --name mysql-db $ecr_dbimage

docker inspect <container_id>
docker inspect de3a09026b53fd03c1d8c3e16340e8e553f5b67083893c94a689c583e20665a1

#Database connection and Show database
docker exec -it mysql-db /bin/bash
mysql -uroot -ppw -e "SHOW DATABASES;"

Exit Sql

// Ip FROM container inspect
export DBHOST=192.168.2.2                 
export DBPORT=3306
export DBUSER=root
export DATABASE=employees
export DBPWD=pw

Pull Image From WebApp Repo
ecr_appimage=141947217581.dkr.ecr.us-east-1.amazonaws.com/clo835-assignment1-app:app-latest

aws ecr get-login-password --region us-east-1 | docker login -u AWS $ecr_appimage --password-stdin

#running containers in detach mode for 
docker run -d -p 8081:8080 -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=blue --network my-bridgenetwork --name blue_app $ecr_appimage

docker run -d -p 8082:8080 -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=pink --network my-bridgenetwork --name pink_app $ecr_appimage

docker run -d -p 8083:8080 -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DATABASE=$DATABASE -e DBPWD=$DBPWD -e APP_COLOR=lime --network my-bridgenetwork --name lime_app $ecr_appimage

Containers can ping each other using their host names. For example, the following should work from inside the blue container: ping pink ping lime
docker exec -it <container-name> /bin/bash
//Execute for every container app
docker exec -it blue_app /bin/bash

#install ping
apt-get update
apt-get install iputils-ping -y
