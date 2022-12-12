# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

![diagram](./images/199484750-f45c1807-4153-4e78-ac17-3d033579e735.png)

## Step One: Configure AWS account and organisational units

Create an AWS Master account. (Also known as Root Account). Within the Root account, create a sub-account and name it DevOps. Search for AWS organisations -> create organisation -> AWS accounts -> add an AWS account. Leave the IAM role name as the default *OrganizationAccountAccessRole *

![account](./images/00.png)

Within the Root account, create an AWS Organization Unit (OU). Name it Dev:

AWS organisations -> AWS accounts -> root -> children -> click actions -> organisational unit -> create new

![OU](./images/000.png)

Move the DevOps account into the Dev OU:

aws organisations -> aws accounts -> select Devops -> under Actions select move
set destination account to dev

Login to the newly created AWS account using the new email address.

Create a domain name for your fictitious company at godaddy.com

Create a hosted zone in AWS:

![hostedzone](./images/01_hostedzone.png)

## Step Two: SET UP A VIRTUAL PRIVATE NETWORK (VPC)

Create a VPC:

![vpc](./images/01_create_vpc.png)

Edit DNS hostnames: click Your VPCs, select a VPC, and choose Actions -> Edit VPC settings -> click Enable DNS hostnames

![enable DNS hostname](./images/02_enable_hostnames.png)

Create An Internet Gateway:

![internet geteway](./images/03_create_internet_gateway.png)

Attach internet gateway to VPC:

![attach vpc](./images/04_attach_vpc.png)

Create 6 subnets as shown in the architecture:

Availability zone a:
    public subnet 1
    private subnet 1
    private subnet 3

Availability zone b:
    public subnet 2
    private subnet 2
    private subnet 4

select subnets from VPC console:

![subnet](./images/05_create_subnets.png)
![subnet1](./images/06_subnet1.png)
![subnet2](./images/07_subnet2.png)
![private1](./images/08_private_subnet1.png)
![private3](./images/10_private_subnet4.png)

Create a route table and associate it with public subnets:

![route table public](./images/11_create_route_table_public.png)

Create a route table and associate it with private subnets:

![route table private](./images/18_private_route_table.png)

on actions button select edit subnet associations and select public subnets:

![edit table](./images/13_edit_public_subnets_associations.png)

on actions button select edit subnet associations and select private subnets:

![edit private](./images/14_edit_private_subnets_associations.png)

Edit a route in public route table, and associate it with the Internet Gateway:

![edit table](./images/15_edit_public_routes.png)

Create 3 Elastic IPs:

![elastic IP](./images/16_elastic_ip.png)

Create a Nat Gateway and assign one of the Elastic IPs (The other 2 will be used by Bastion hosts):

![nate gateway](./images/17_create_nat_gateway.png)

Edit a route in private route table, and associate it with the Nat Gateway:

![edit route table](./images/18_private_route_table.png)

Create a Security Group for:

1) Application Load Balancer: ALB will be available from the Internet which will be name external ALB

![ext ALB](./images/19_create__ext_security_group.png)

2) Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address

![bastion sec group](./images/20_create_bastion_security_group.png)

3) Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB)

![nginx](./images/21_nginx_security_group.png)

4) Internal ALB:

![internal sec group](./images/22_ALB_int_security_group.png)

5) Web Servers: Access to Webservers should only be allowed from the Nginx servers

![web server sec group](./images/23_webserver_security_group.png)

6) Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS). Only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![Data layer sec](./images/24_security_group_datalayer.png)

## Step Three:Setup Compute Resources

TLS Certificates From Amazon Certificate Manager

Request a public wildcard certificate for the domain name you registered. In certificate manager console, click request certificate and then add * and domain name * with  validation maethod: DNS validation.

![cert](./images/25_request_public_certificate.png)
![cert](./images/26_request_certificate.png)

3 EC2 Instance based on Red Hat of the t2.micro family were launched for nginx, bastion and the one for the two webservers

Set Up Compute Resources for Bastion

Ensure that it has the following software installed:

python
ntp
net-tools
vim
wget
telnet
epel-release
htop

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd 

systemctl enable chronyd
```

Set Up Compute Resources for Nginx

Ensure that it has the following software installed:

python
ntp
net-tools
vim
wget
telnet
epel-release
htop

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

Configure selinux policies for Nginx servers:

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```

Install amazon efs utils for mounting the target on the Elastic file system:

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```

Seting up self-signed certificate for the nginx instance:

```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/Kebs.key -out /etc/ssl/certs/Kebs.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

![self](./images/40_self_signed_cert.png)

Set Up Compute Resources for Websevers:

```
sudo su -

yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

Configure selinux policies for web servers:

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```

Install amazon efs utils for mounting the target on the Elastic file system:

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```

Setting up self-signed certificate for the apache webserver instance:

```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/Kebs.key -x509 -days 365 -out /etc/pki/tls/certs/Kebs.crt

vi /etc/httpd/conf.d/ssl.conf
```

![ssl conf](./images/43_edit_ssl_conf.png)

Create an AMI out of the EC2 instances:

In EC2 dashboard, select nginx -> actions -> image and templates -> create image

![ami](./images/44_create_image_webserver.png)

Configure Target Groups:

![target groups](./images/45_create_nginx_target_group.png)

Configure Applipcation Load balancers (ALB) (one External, One Internal)
Application Load Balancer To Route Traffic To NGINX

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

![load balancer](./images/46_create_load_balancer_mapping.png)

Create an Internet facing ALB
Ensure that it listens on HTTPS protocol (TCP port 443)
Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
Select Security Group
Select Nginx Instances as the target group

![load balancer](./images/46_create_load_balancer_target_group.png)

Ensure that it listens on HTTPS protocol (TCP port 443)
Ensure the ALB is created within the appropriate VPC | AZ | Subnets
Choose the Certificate from ACM
Select Security Group
Select webserver Instances as the target group
Ensure that health check passes for the target group

![rule](./images/47_add_rule.png)

Create Launch Templates

Bastion Launch Template:

![template](./images/48_create_launch_template1.png)
![bastion](./images/48_create_launch_template2.png)

Bastion Launch Template User Data:

```
#!/bin/bash 
yum install -y mysql 
yum install -y git tmux 
yum install -y ansible
```

Ngnix Launch Template and user Data:

![nginx](./images/48_create_launch_template4_nginx_userdata.png)

Wordpress Launch Template:

![wordpress](./images/48_create_launch_template4_wordpress_userdata.png)

Tooling launch Template and user Data:

![tooling template](./images/49_launch_template_tooling.png)

Configure Auto Scaling Groups:

Configure Autoscaling group For Bastion

![autoscaling](./images/50_create_auto_scaling.png)

Configure Autoscaling Group For Ngnix:

![autoscaling](./images/51_auto_scaling_group_nginx.png)

Configure Autoscaling Group For Wordpress:

Create wordpressdb and toolingdb on Kebs-Bastion launch template

![db](./images/52_create_dbs.png)

Checks on wordpress server:

![health](./images/54.png)

