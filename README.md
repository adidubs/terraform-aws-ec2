# terraform-aws-ec2
1. Introduction

This project demonstrates how to provision an EC2 instance in Amazon Web Services using Infrastructure as Code (IaC) with Terraform.

The goal is to create a simple, industry-standard Terraform project structure that can be used to:

Understand Terraform fundamentals

Learn file structure used in real DevOps environments

Create infrastructure in a repeatable and automated way

The infrastructure created in this demo:

1 EC2 Instance using Amazon EC2

Latest Amazon Linux 2 AMI

Instance type: t2.micro

Region: us-east-1 

2. Prerequisites

Before running this project, ensure the following tools are installed.

1. Terraform

Verify installation:

terraform -version

Terraform is used to define and provision infrastructure through code.

2. AWS CLI

Install AWS CLI and verify:

aws --version

The AWS CLI is used to authenticate Terraform with AWS.

3. AWS Account

You must have an active Amazon Web Services account and IAM credentials.

3. Project Structure

The project follows an industry-standard Terraform folder structure.

terraform-project/
│
├── main.tf
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf

Explanation of each file:

File	Purpose
provider.tf	Defines Terraform and AWS provider configuration
main.tf	Contains infrastructure resources
variables.tf	Declares variables used in the project
terraform.tfvars	Provides values for variables
outputs.tf	Displays outputs after infrastructure is created

Terraform automatically loads all .tf files in the directory and treats them as one combined configuration.

4. Provider Configuration

File: provider.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  required_version = ">= 1.0"
}

provider "aws" {
  region = var.aws_region
}
Explanation

The terraform block specifies:

The required provider

The provider source

Compatible version

The provider block configures the AWS connection and defines the region where infrastructure will be deployed.

The region value is taken from a variable to avoid hardcoding configuration.

5. Variables Declaration

File: variables.tf

variable "aws_region" {
  description = "AWS region for resources"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "instance_name" {
  description = "Name of EC2 instance"
  type        = string
}
Explanation

Variables allow Terraform code to be reusable and configurable.

Instead of hardcoding values, variables allow the same code to be used in different environments.

Example:

Dev → t2.micro
Prod → t3.medium
6. Variable Values

File: terraform.tfvars

aws_region    = "ap-south-1"
instance_type = "t2.micro"
instance_name = "terraform-demo-instance"
Explanation

This file assigns values to variables defined in variables.tf.

Terraform automatically loads .tfvars files when running commands.

7. Infrastructure Definition

File: main.tf

data "aws_ami" "amazon_linux" {

  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

}

resource "aws_instance" "web_server" {

  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }

}
7.1 Data Source
data "aws_ami" "amazon_linux"

A data source fetches existing information from AWS instead of creating new resources.

In this case, Terraform retrieves the latest Amazon Linux 2 AMI.

Important configuration:

Parameter	Purpose
most_recent	Retrieves the latest AMI
owners	Ensures AMI belongs to Amazon
filter	Matches specific AMI pattern
7.2 Resource Block
resource "aws_instance" "web_server"

This block creates an EC2 instance.

Structure of resource blocks:

resource "<provider_resource>" "<local_name>"

Example:

aws_instance → AWS EC2 resource
web_server → Local Terraform name
7.3 Instance Configuration
AMI
ami = data.aws_ami.amazon_linux.id

The instance uses the AMI returned by the data source.

Instance Type
instance_type = var.instance_type

The instance type is provided through variables.

Tags
tags = {
  Name = var.instance_name
}

AWS does not have a dedicated instance name field.
The name visible in the AWS console is actually a tag with key Name.

8. Outputs

File: outputs.tf

output "instance_id" {
  description = "ID of EC2 instance"
  value       = aws_instance.web_server.id
}

output "instance_public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.web_server.public_ip
}
Explanation

Outputs allow Terraform to display useful information after deployment.

Example output:

instance_id = i-0abcd123456
instance_public_ip = 13.xxx.xxx.xxx

These outputs can be used for:

SSH access

CI/CD pipelines

Monitoring systems

9. AWS Authentication

Terraform requires credentials to interact with AWS.

The recommended method is using the AWS CLI.

Run:

aws configure

Provide:

AWS Access Key ID
AWS Secret Access Key
Default region
Output format

Credentials are stored in:

C:\Users\<username>\.aws\credentials

Terraform automatically reads these credentials.

10. Running Terraform

Navigate to the project directory and execute the following commands.

Step 1 – Initialize Terraform
terraform init

This command:

Downloads provider plugins

Prepares Terraform working directory

Step 2 – Generate Execution Plan
terraform plan

This shows what Terraform will create.

Example output:

+ aws_instance.web_server

No resources are created at this stage.

Step 3 – Apply Configuration
terraform apply

Terraform will:

Retrieve the latest AMI

Create an EC2 instance

Apply tags

Display outputs

11. Infrastructure Created

After successful deployment, the following resource will exist in AWS:

Resource	Value
Instance	EC2
OS	Amazon Linux 2
Instance Type	t2.micro
Region	ap-south-1
Name Tag	terraform-demo-instance
12. Important Terraform Concepts
Terraform State

Terraform keeps track of infrastructure in a state file:

terraform.tfstate

This file maps Terraform configuration to real cloud resources.

Dependency Graph

Terraform automatically determines resource dependencies.

Example:

AMI Data Source
      ↓
EC2 Instance

The AMI must be retrieved before creating the instance.

13. Cleaning Up Resources

To destroy the infrastructure:

terraform destroy

Terraform will remove all resources defined in the configuration.
