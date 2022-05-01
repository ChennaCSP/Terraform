# Terraform Sample templates
# main.tf -file

provider "aws" {
  profile = "default"
region = "us-east-1"
}




# key- creation
resource "tls_private_key" "tls_key" {
  algorithm = "RSA"
}

resource "aws_key_pair" "generated_key" {
  key_name    = "mykey"
  public_key    = "${tls_private_key.tls_key.public_key_openssh}"

  depends_on  = [
    tls_private_key.tls_key
  ]
}

resource "local_file" "key-file" {
  content  = "${tls_private_key.tls_key.private_key_pem}"
  filename = "mykey.pem"

  depends_on = [
    tls_private_key.tls_key
  ]
}



# resource instance

resource "aws_instance" "myFirstInstance" {
  ami           = "ami-0b9064170e32bde34"
key_name = “mykey”
instance_type = var.instance_type
security_groups= [ "SEC Security Group"]
  tags= {
    Name = "SEC Security Group"
  }
}
 


# Creating a security group for 80/443 port Number
 
# Terraform AWS Security Group
resource “aws_security_group” “SEC-Security-Group”
{
name = ”SEC Security Group” 
description  = “Enable HTTP/HTTPS access on Port 80/443”
vpc_id = SEC_aws_vpc.vpc.id


# ingress rule
ingress
{
description = “HTTP Access”
from_port  = 80
to_port = 80
protocol  = “tcp”
cidr_blocks = [“0.0.0.0/0”]
}

ingress
{
description = “HTTP Access”
from_port  = 443
to_port = 443
protocol  = “tcp”
cidr_blocks = [“0.0.0.0/0”]
}

egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
  }


#tags =
{
Name = “SEC Security Group”
     }
}

#Main VPC 
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/18"

  tags = {
    Name = "Main VPC"
  }
}

# Public Subnet with Default Route to Internet Gateway

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.0.0/24"

  tags = {
    Name = "Public Subnet"
  }
}


# Private Subnet with Default Route to NAT Gateway

resource "aws_subnet" "private" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "Private Subnet"
  }
}


# Main Internal Gateway for VPC


resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "Main IGW"
  }
}



# Elastic IP for NAT Gateway

resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.igw]
  tags = {
    Name = "NAT Gateway EIP"
  }
}


# Main NAT Gateway for VPC

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public.id

  tags = {
    Name = "Main NAT Gateway"
  }
}


# Route Table for Public Subnet

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "Public Route Table"
  }
}



# Association between Public Subnet and Public Route Table

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}



# Route Table for Private Subnet

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "Private Route Table"
  }
}



# Association between Private Subnet and Private Route Table

resource "aws_route_table_association" "private" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private.id
}

Terraform Template(s)
For AWS 
Basic for creating Security Group
Reference - official Terraform
module "security-group" {
  source  = "figurate/security-group/aws"
  version = "1.0.5"
  # insert the 2 required variables here
}


another**
resource "aws_security_group_rule" "example" { type = "ingress" from_port = 0 to_port = 65535 protocol = "tcp" cidr_blocks = [aws_vpc.example.cidr_block] ipv6_cidr_blocks = [aws_vpc.example.ipv6_cidr_block] security_group_id = "sg-123456" }



Basic Usage
resource "aws_security_group" "allow_tls" {
 name = "allow_tls" 
description = "Allow TLS inbound traffic" 
vpc_id = aws_vpc.main.id ingress = 
[
 { description = "TLS from VPC" 
from_port = 443 
to_port = 443
 protocol = "tcp" 
cidr_blocks = [aws_vpc.main.cidr_block]
 ipv6_cidr_blocks = [aws_vpc.main.ipv6_cidr_block]
 } 
] 
egress = [
 { 
from_port = 0
 to_port = 0 
protocol = "-1" 
cidr_blocks = ["0.0.0.0/0"] 
ipv6_cidr_blocks = ["::/0"] } ]
 tags = { Name = "allow_tls" } }

For other config…. 
resource "aws_security_group"
 "example" { # ... other configuration ... 
egress = [ { from_port = 0
 to_port = 0
 protocol = "-1"
 cidr_blocks = ["0.0.0.0/0"] 
ipv6_cidr_blocks = ["::/0"] } ] }


resource "aws_security_group" 
"example" { # ... other configuration ... 
egress = [ { 
from_port = 0 
to_port = 0
 protocol = "-1"
 prefix_list_ids = [aws_vpc_endpoint.my_endpoint.prefix_list_id] } ] } 
resource "aws_vpc_endpoint" "my_endpoint" { # ... other configuration ... }




Reference - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group

A Secure Cloud
 
Securing a S3 bucket -- with versioning
module "s3_bucket" {
source = "terraform-aws-modules/s3-bucket/aws"

bucket = "my-s3-bucket"
acl = "private"

versioning = {
enabled = true
}
}
resource "aws_security_group" "wp_sg" {
  name        = "wp-SG"
  vpc_id      = "${aws_vpc.itsvpc.id}"

Creating Security Group for port no. 22 -ssh & 80--tcp thats HTTP
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
For port 22 and 3306
resource "aws_security_group" "mysql_sg" {
  name        = "sqlSG"
  vpc_id      = "${aws_vpc.itsvpc.id}"
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    security_groups = [aws_security_group.wp_sg.id]
  }
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    security_groups = [aws_security_group.wp_sg.id]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
3.

resource "aws_security_group" "wp_sg" {
  name        = "wp-SG"
  vpc_id      = "${aws_vpc.itsvpc.id}"


  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["1.2.3.4"]
  }




  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [" 23.4.5.6"]
  }
}

Scenario:
inside a vpc : Create one instance and the create a security group and security rules 80 and 443 inside and a outside web server 


terraform {
  required_version = "~> 0.13"
  required_providers {
    aws = {
        source = "hashicorp/aws"
        versions = "~>0.3"
 
     }
  }
}
 
 
provider "aws" {
  profile = "default"
  region  = "us-east-2"
}
 
resource "aws_instance" "ec2vm" {
  ami-""
  instance_type="t3.micro"
}
 
module "security-group" {
  source  = "figurate/security-group/aws"
  version = "1.0.5"
  
}
