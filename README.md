# clo835_fall2024_assignment1_app
# Below is the instruction to pull image from repo and run containers on a newly provisioned EC2 instance
# Install and enable Docker
sudo yum update -y
sudo yum install docker
sudo service docker start
sudo usermod -a -G docker ec2-user
# If we add ec2-user to the docker group, need to start a new session for the change to take effect
# AWS configuration
aws configure set aws_access_key_id <>
aws configure set aws_secret_access_key <>
aws configure set aws_session_token <>
# login and pull image, example of repo uri: 444868690040.dkr.ecr.us-east-1.amazonaws.com
export ECR={your repository uri}/clo835-assignment1-mysql-repo
export ECR={your repository uri}/clo835-assignment1-webapp-repo
aws ecr get-login-password --region us-east-1 | docker login -u AWS ${ECR} --password-stdin
docker pull {your repository uri}/clo835-assignment1-mysql-repo:v0.1
docker pull {your repository uri}/clo835-assignment1-webapp-repo:v0.1
docker tag {your repository uri}/clo835-assignment1-mysql-repo:v0.1 mysql-db-image
docker tag {your repository uri}/clo835-assignment1-webapp-repo:v0.1 webapp-image
# Create a new network
docker network create --driver bridge my_custom_bridge
# MySQL containers
docker run -d --name mysql-db --network my_custom_bridge -e MYSQL_ROOT_PASSWORD=pw mysql-db-image
# Test MySQL container
docker exec -it mysql-db sh
mysql -u root -p
SHOW DATABASES;
exit
exit
# Create WebApp containers
docker inspect mysql-db
export DBHOST=172.18.0.2
export DBPORT=3306
export DBUSER=root
export DATABASE=employees
export DBPWD=pw
export APP_COLOR1=blue
export APP_COLOR2=pink
export APP_COLOR3=lime
docker run -d -p 8081:8080 --name blue --network my_custom_bridge -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR1 webapp-image
docker run -d -p 8082:8080 --name pink --network my_custom_bridge -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR2 webapp-image
docker run -d -p 8083:8080 --name lime --network my_custom_bridge -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e DBUSER=$DBUSER -e DBPWD=$DBPWD -e APP_COLOR=$APP_COLOR3 webapp-image
# Check ping between containers
docker exec -it blue_container sh
apt-get install -y iputils-ping
ping pink
ping lime
