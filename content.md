## Installing docker: 

Run an EC2 instance using Linux2 AMI and open port 8080 to internet 

sudo yum update -y

sudo amazon-linux-extras install docker -y

sudo service docker start -y (or) systemctl start docker 

systemctl status docker

systemctl enable docker --> running docker on startup

sudo usermod -a -G docker ec2-user --> ec2-user can execute docker commands without using sudo

docker info

source: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

## Installing docker-compose:

sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose  --> copy latest docker-compose binary from GitHub

sudo chmod +x /usr/local/bin/docker-compose --> Give executable permission to the folder

docker-compose version --> check

## Jenkins:

docker pull jenkins/jenkins

docker images

docker ps -a 

create a directory /home/jenkins/jenkins-data and a docker-compose.yml file as below:

version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - 8080:8080
    volumes:
      - "$PWD/jenkins_home:/var/jenkins_home"
    networks:
      - net
networks:
   net:
   
sudo chown 1000:1000 enkins_home -R --> make sure jenkins user can write to the volumes directory as once the server is terminated all the data is lost. 'R' means apply these changes recurrsively

cat /etc/passwd or id --> list of all users

docker-compose up -d --> executes the contents of compose file

docker-compose down 

docker-compose start --> similar to the above command

docker-compose stop --> stops the service

docker logs -f jenkins --> logs with the password hash to login to jenkins

Install suggested plugins and create a admin user 











