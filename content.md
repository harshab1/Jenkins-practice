## Installing docker: 

Run an EC2 instance using Linux2 AMI

sudo yum update -y

sudo amazon-linux-extras install docker

sudo service docker start (or) systemctl start docker 

systemctl status docker

systemctl enable docker --> running docker on startup

sudo usermod -a -G docker ec2-user --> ec2-user can execute docker commands without using sudo

docker info

source: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

## Installing docker-compose:

sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose  --> copy latest docker-compose binary from GitHub

sudo chmod +x /usr/local/bin/docker-compose --> Give executable permission to the folder

docker-compose version --> check



