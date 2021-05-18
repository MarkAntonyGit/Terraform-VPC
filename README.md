# Terraform-VPC
To use terraform to create a AWS, VPC with desired CIDR and with public and private subnets.

## Description 

[Terraform](https://www.terraform.io/intro/) is an infrastructure as a code (IAAC) tool created by [HashiCorp](https://www.hashicorp.com/) which can be used for managing infrastructure with various technologies like Amazon AWS, Microsoft Azure, Google Cloud, and vSphere etc. 

Here we are using Terraform for creating a [VPC](https://aws.amazon.com/vpc/) with private/public Subnet and Network Gateway's for the VPC. We will be making 1 VPC with 6 Subnets: 3 Private and 3 Public, 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables. All these resource creation can be automated with desired values and you can use this whenever you need to create a VPC. 
