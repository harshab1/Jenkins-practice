Run an EC2 instance using Linux2 AMI

sudo yum update -y

sudo amazon-linux-extras install docker

sudo service docker start (or) systemctl start docker 

systemctl status docker

systemctl enable docker --> running docker on startup

sudo usermod -a -G docker ec2-user --> ec2-user can execute docker commands without using sudo

docker info

