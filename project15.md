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

