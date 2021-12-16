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

mkdir -p /home/jenkins/jenkins-data --> creates folders and sub-folders recursively

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

docker run image-id --> to run docker container from a image

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
    echo "remote_user:password" | chpasswd && \
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

create database:

connect to the mysql db from the remote_host: mysql -u root -h db_host -p and enter password

create database db_name;

use db_name;

create table table_name(col1, col2, col3..);

insert into table_name values (val11, val12, val13..), (val_21, val22, val23..);

select * from table_name;

desc table_name; 

create a S3 bucket on AWS

create IAM user with programatic access and S3FullAccess policy

Taking manual backups:

login to remote_host:

mysqldump -u root -h db_host -p testdb > /tmp/db.sql

export AWS_ACCESS_KEY_ID=value

export AWS_SECRET_ACCESS_KEY=value

aws s3 cp /tmp/db.sql s3://bucket-name/remote-file-name.sql

-- #/bin/bash  --> remove '--'

DATE=$(date +%H-%M-%S)
BACKUP=db-$DATE.sql

DB_HOST=$1
DB_PASSWORD=$2
DB_NAME=$3
AWS_SECRET=$4
BUCKET_NAME=$5

mysqldump -u root -h $DB_HOST -p$DB_PASSWORD $DB_NAME > /tmp/$BACKUP && \
export AWS_ACCESS_KEY_ID=AKIAJRWZWY3CPV3F3JPQ && \
export AWS_SECRET_ACCESS_KEY=$AWS_SECRET && \
echo "Uploading your $BACKUP backup" && \
aws s3 cp /tmp/db-$DATE.sql s3://$BUCKET_NAME/$BACKUP

manage sensitive data like keys and passwords using jenkins credentials


## Jenkins and Ansible:

Automate the process of fetching information from a database and display the output in a tabular format on a webpage. 

Automate - Jenkins
Get the relevant information from DB - Ansible

## Jenkins Security:

Configure the user permissions with role plugin. Manage the roles with permissions for each detail and also for the jobs with a pettern using '.*'

## Jenkins Tips & Tricks:

Jenkins global variables are default values set by Jenkins, could be used to identify a job/ build. We can our own custon global variables from configure section.

Modify the Jenkins URL if there is a real DNS assciated with the server.

Execute Jenkins jobs automatically using cron expression

Trigger a Jenkins job from external user: create a user with the required permissions

Execute a job from script: 

without parameters:

pass the build now url link with crumbs header 

with parameters:

same as above, also pass parameters in the url of the script

## Jenkins and Email:

plugin 'mailer' is installed

Integrate Jenkins with Amazon SES

Without SES, emial configuration can be done from Jenkins directly

In the post build options provide the email of the recepient

## Jenkins and maven:

Maven Integration plugin has to be installed

Install GIT plugin

Put the git clone URL in the Source Code Management section of the job configuration

Workspace contains the code contents of Jenkins

Look for maven goals, they have a meaning. They are defined in the code to execute certain tasks like testing the jar

Use execute shell to run commands on the jar file built by maven

Trend graph can be generated showing the success and failure trends of the jobs. It is set in Junit tests in post build actions

Archiving the artifact is post build option, that is the result of execution of a certain job, it can be downloaded from that job's homepage. And better the artifact is archived if only the buid is successful.

Email notifications can also be setup

## Jenkins and GIT

set up gitlab

Gitlab is similar to git. The same functionality of source code management. Jenkins job can be set in a similar way to git 

Git hook can be used to trigger a action on a commmit to a specific branch using a custom script

## Jenkins DSL

DSL is installed using jsl plugin, is used to create Jenkins job using script written in Ruby language

Script sample example can be taken from: https://jenkinsci.github.io/job-dsl-plugin/ . Always prefer free-style job sample examples.

The contents of the DSL file could be: Description, Parameters, SCM, Triggers (like cron job or pollscm), Steps (shell execution), Mailer, etc.

The DSL code could be versioned using git















