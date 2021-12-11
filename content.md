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
   
sudo chown 1000:1000 jenkins_home -R --> make sure jenkins user can write to the volumes directory as once the server is terminated all the data is lost. 'R' means apply these changes recurrsively

cat /etc/passwd or id --> list of all users

docker-compose up -d --> executes the contents of compose file

docker-compose down 

docker-compose start --> similar to the above command

docker-compose stop --> stops the service

docker logs -f jenkins --> logs with the password hash to login to jenkins

Install suggested plugins and create a admin user 

docker exec -ti container-name bash --> to run commands with in a container

docker cp file-name container-name:full-path --> used to copy a file in to a specific container. Using volumes is better

parameters --> Build parameters are used to pass data to Jenkins job (https://www.baeldung.com/ops/jenkins-parameterized-builds)

## Jenkins & Docker

Dockerfile is used to create a image from an pre-existing image that is used to build a container

mkdir centos 

vi Dockerfile
--

FROM centos:7

RUN yum -y install openssh-server

RUN useradd remote_user && \
    echo "password" | passwd remote_user --stdin && \
	mkdir /home/remote_user/.ssh && \
	chmod 700 /home/remote_user/.ssh
	
COPY remote-key.pub /home/remote_user/.ssh/authorized_keys --> ssh-keygen -f remote-key

RUN chown remote_user:remote_user /home/remote_user/.ssh/ -R && \
    chmod 600 /home/remote_user/.ssh/authorized_keys
	
RUN ssh-keygen -A --> to create global keys for SSH

RUN yum -y install mysql

RUN yum -y install epel-release && \
    yum -y install python3-pip && \
    pip3 install --upgrade pip && \
    pip3 install awscli

CMD /usr/sbin/sshd -D --> command for what docker has to run

Adding the new service:

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
  remote_host:
    container_name: remote_host
	image: remote_host
	build:
	  context: centos7  --> location for building image
	networks:
	  - net     --> Makes sure the jenkins server and this service is on same network
networks:
   net:
   
   
docker-compose build --> will build the image based on context provided inbuild section

docker-compose up -d

docker cp remote-key jenkins:/tmp/remote-key --> ssh -i remote-key remote_user@remote_host

Install SSH plugin on Jenkins and restart 

Manage Jenkins > Manage Credentials > Jenkins > Global credentials > Add credentials > Enter remote user and private remote-key 

Manage Jenkins > Configure system > SSH site > remote_host , Port 22 and remote_user credentials and check the connection

Create a job with Build option as Execute shell script on remote host using SSH

## MySQL + AWS + Shell Scripting + Jenkins:

The task is to automate taking the MySQL backup into Amazon S3 Bucket

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
  remote_host:
    container_name: remote_host
	image: remote_host
	build:
	  context: centos7  
	networks:
	  - net
  db_host:
    container_name: db
	image: mysql:5.7
	environment:
	  - "MYSQL_ROOT_PASSWORD = password"
	volumes:
	  - "$PWD/db_data:/var/lib/mysql"
	networks:
	  - net
networks:
   net:
   
docker exec -ti db bash

mysql -u root -p  --> to login into mysql with the password

remote host need mysql and awscli installed --> the above Dockerfile is updated with all the required RUN commands









