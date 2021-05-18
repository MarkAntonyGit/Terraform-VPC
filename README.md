# Terraform-VPC
To use terraform to create a AWS, VPC with desired CIDR and with public and private subnets.

## Description 

[Terraform](https://www.terraform.io/intro/) is an infrastructure as a code (IAAC) tool created by [HashiCorp](https://www.hashicorp.com/) which can be used for managing infrastructure with various technologies like Amazon AWS, Microsoft Azure, Google Cloud, and vSphere etc. 

Here we are using Terraform for creating a [VPC](https://aws.amazon.com/vpc/) with private/public Subnet and Network Gateway's for the VPC. We will be making 1 VPC with 6 Subnets: 3 Private and 3 Public, 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables. All these resource creation can be automated with desired values and you can use this whenever you need to create a VPC. 

### Features
- Easy to customize and use as the Terraform modules are created using variables,allowing the module to be customized without altering the module's own source code, and allowing modules to be shared between different configurations.
- Each subnet CIDR block created  automatically using cidrsubnet Function (Automated)
- AWS informations are defined using tfvars file and can easily changed (Automated/Manual)
- Project name is appended to the resources that are creating which will make easier to identify the resources. 

### Prerequisites for this project
- Need a IAM user access with attached policies for the creation of VPC.
- Knowledge to the working principles of each AWS services especially VPC, EC2 and IP Subnetting.

### How to use?

##### To install Terraform

```sh
wget https://releases.hashicorp.com/terraform/0.15.3/terraform_0.15.3_linux_amd64.zip
unzip terraform*.zip
mv -f terraform /usr/bin/
```

##### To execute this terraform code

```sh
yum install git unzip -y
git clone https://github.com/MarkAntonyGit/Terraform-VPC
cd Terraform-VPC
vim terraform.tfvars - Make Necessary Changes.
terraform validate
terraform plan
terraform apply
```
> screenshots
- _Tfvars File_

![](https://i.ibb.co/wW7s7cR/tfvars.jpg)
- _Plan_

![](https://i.ibb.co/LRtnTRR/plan.jpg)
- _State List_

![](https://i.ibb.co/4tSNnfm/state.jpg)

# Behind the script
# Terraform Installation and VPC creation code and explanation :
> ##### please note that you can use the below steps to make a vpc through terraform (_Manually_).

### Prerequisites 
- Create an IAM user on your AWS console that have access to create the required resources.
- Create a dedicated directory where you can create terraform configuration files.

> sample installation steps
```sh
wget https://releases.hashicorp.com/terraform/0.15.3/terraform_0.15.3_linux_amd64.zip
unzip terraform_0.15.3_linux_amd64.zip 
ls -l
-rwxr-xr-x 1 root root 79991413 May  6 18:03 terraform  <<=======
-rw-r--r-- 1 root root 32743141 May  6 18:50 terraform_0.15.3_linux_amd64.zip
mv terraform /usr/bin/
which terraform 
/usr/bin/terraform
```

Lets create a file for declaring the variables.This is used to declare the variable and the values are passing through the terrafrom.tfvars file.

### Create a variables.tf file
```sh
variable "region" {}
variable "access_key" {}
variable "secret_key" {}
variable "vpc_cidr" {}
variable "project" {}
```
### Create a provider.tf file 
```sh
provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```
### Create a terraform.tfvars
By default terraform.tfvars will load the variables to the the reources.
You can modify accordingly as per your requirements.

```sh
region     = "put-your-region-here"
access_key = "put-your-access-key-here"
secret_key = "put-your-secret-key-here"
project = "name-of-your-project"
vpc_cidr = "X.X.X.X/X"
```
Go to the directory that you wish to save your tfstate files.Then Initialize the working directory containing Terraform configuration files using below command.
```sh
terraform init
```
#### Lets start creating main.tf file with the details below.


> To create VPC
```sh
resource "aws_vpc" "vpc" {
  cidr_block           = var.vpc_cidr
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "${var.project}-vpc"
  }
}
```
> To Gather All Subent Name
```sh
data "aws_availability_zones" "available" {
  state = "available"
}
```

> To create InterGateWay For VPC
```sh
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = "${var.project}-igw"
  }
}
```
Here in this infrastructre we shall create 3 public and 3 private subnets in the region.This sample was meant for regions having 6 availability zone. I have used "us-east-1". Choose your region and modify according to the availability of the AZ. Also we have already provided the CIDR block in our terraform.tfvars you dont need to calculate the subnets, here we use terraform to automate the subnetting in /19.

> Creating public1 Subnet
```sh
resource "aws_subnet" "public1" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 0)
  availability_zone        = element(data.aws_availability_zones.available.names,0)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public1"
  }
}
```
> Creating public2 Subnet
```sh
resource "aws_subnet" "public2" {
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 1)
  availability_zone        = element(data.aws_availability_zones.available.names,1)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public2"
  }
}
```
> Creating public3 Subnet
```sh
resource "aws_subnet" "public3" {
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 2)
  availability_zone        = element(data.aws_availability_zones.available.names,2)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public-3"
  }
}
```
> Creating private1 Subnet
```sh
resource "aws_subnet" "private1" {
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 3)
  availability_zone        = element(data.aws_availability_zones.available.names,3)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private1"
  }
}
```
> Creating private2 Subnet
```sh
resource "aws_subnet" "private2" {
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 4)
  availability_zone        = element(data.aws_availability_zones.available.names,4)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private2"
  }
}
```
> Creating private3 Subnet
```sh
resource "aws_subnet" "private3" {
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 5)
  availability_zone        = element(data.aws_availability_zones.available.names,5)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private3"
  }
}
```
> Creating  Elastic IP For Nat Gateway
```sh
resource "aws_eip" "eip" {
  vpc      = true
  tags     = {
    Name = "${var.project}-eip"
  }
}
```
> Attaching Elastic IP to NAT gateway
```sh
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.eip.id
  subnet_id     = aws_subnet.public1.id
  tags = {
    Name = "${var.project}-nat"
  }
}
```
>  Creating Public Route Table
```sh
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = {
    Name = "${var.project}-public"
  }
}
```
>  Creating Private Route Table
```sh
  resource "aws_route_table" "private" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
  tags = {
    Name = "${var.project}-private"
  }
}
```
> Creating Public Route Table Association
```sh
resource "aws_route_table_association" "public1" {        
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}
resource "aws_route_table_association" "public2" {      
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}
resource "aws_route_table_association" "public3" {       
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}
```
> Creating Private Route Table Association
```sh
resource "aws_route_table_association" "private1" {        
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}
resource "aws_route_table_association" "private2" {      
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}
resource "aws_route_table_association" "private3" {       
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}
````
> Create an output.tf for getting  terrafrom output.
```sh
output "aws_eip" {
value = aws_eip.eip.public_ip
}
output "aws_vpc" {
value = aws_vpc.vpc.id
}
output "aws_internet_gateway" {
value = aws_internet_gateway.igw.id
}
output "aws_nat_gateway" {
value = aws_nat_gateway.nat.id
}
output "aws_route_table_public" {
value = aws_route_table.public.id
}
output "aws_route_table_private" {
value = aws_route_table.private.id
}
```
#### Lets validate the terraform files using
```sh
terraform validate
```
#### Lets plan the architecture and verify once again.
```sh
terraform plan
```
#### Lets apply the above architecture to the AWS.
```sh
terraform apply
```
