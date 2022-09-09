# Terraform

![image](https://user-images.githubusercontent.com/47668244/189112235-fc2275cf-ed0f-4dbc-b7d8-76e1537c4f38.png)

Terraform is a tool created for facilitating infrastructure as code (IaC), by codifying applicaiton program interfaces (APIs) into declarative configuration files.

Hashicorp - open source

## Use Cases

#### Adopt

#### Build




#### Standardize

#### Innovate

#### Multi-cloud deployment

#### Manage Kubernetes

#### Manage virtual machine images

#### Manage network infrastructure

#### Integrate with Existing Workflows
Such as existing CI/CD deployments

## Terraform vs Ansible

main.tf is not reusable compared to an ansible playbook, therefor no very scalable either

### Chocolatey

Chocolatey is software management automation for Windows wrapping necessary resources into compiled packages. This is what we will be using to download Terraform on Windows.

![image](https://user-images.githubusercontent.com/47668244/189112312-7010cb57-cb9f-4f5a-954a-55dfd91ce6c6.png)

Chocolatey has the largest online registry of Windows packages. Chocolatey packages encapsulate everything required to manage a particular piece of software into one deployument artifact - by wrapping installers, executables, zips and/or scripts into a compiled package file.

#### Installation

Go to Chocolatey install page on the website > select appropriate installation solution.

FOr windows this requires entering and executing the following command in powershell: 
```
`Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))`
```
You can then check the version & status of chocolatey with `choco`

#### Installing Terraform

To install Terraform, you must first have the prerequesite dependancy of Chocolatey. Once done, in the powershell: `choco install terraform`

Let it run, and check with `terraform --version`

(if you're using VSCode) Download the vscode extension/package for Hashicorp Terraform. Note also, that using chocolatey & terraform on VSCode terminal doesn't always work, so alternates are Git Bash (admin) and Powershell (admin).

.gitignore all your terraform files when uploading/pushing to Github or elsewhere.

## Launching with Terraform

### AWS Access Keys

`Windows Key` type `env` 'edit system environment variable should appear > click `environment variables` > in `User variables`, click `New...` > enter the name and value for your secret & access keys. Names should be as;
- `AWS_ACCESS_KEY_ID` for the value get and enter your aws access key value
- `AWS_SECRET_ACCESS_KEY` for the value get and enter your aws secret key value

#### Import

Key Pairs can be imported using the key_name, e.g.,

- `terraform import aws_key_pair.deployer deployer-key`

### Main.tf

make a `main.tf` file and configure script with:

```
# who is the cloud provider
# it's aws
provider "aws" {

# within the cloud which part of the world
# we cant to use eu-west-1
    region = "eu-west-1"
}

# init and download required packages
# terraform init

# create a block of code to launch ec2-server
# which resources do we like to create

resource "aws_instance" "app_instance" {

# using which ami
    ami = "ami-0b47105e3d7fc023e" # ubuntu 18.04

# instance type
    instance_type = "t2.micro"

# instance subnet
    subnet_id = aws_subnet.public-subnet-1.id

# do we need it to have public ip [we do]
    associate_public_ip_address = true

# security group
    #vpc_security_group_ids = aws_security_group.allow-ssh.id

# how to name your instance
    tags = {
        Name = "eng122_christian_terraform_app"
    }

    # find out how to attach your file.pem
    key_name = "eng122_christian_terraform"
}
```

(`NOTE!` Please note that the lines defining the subnet and security groups will not work unless you have a vpc.tf file making a vpc and defining each thing respectively. So if you don't have this, feel free to take those lines out of the above code when coppying.)

then `terraform init` in the directory that you've made the 'main.tf' file

- `terraform plan` checks code, always do first to check your code before actually launching whatever your doing. Using the whole 'measure twice, cut one' methodology
- `terraform apply` to launch your instance, as configured in your main.tf file.

- `terraform destloy` to delete your instance

 REMEMBER!: Don't forget to type `yes` in the commandline, when prompted (typically with Terraform plan & terraform apply).

 So by doing this, we've codified, automated the launching of services on AWS, thanks to Terraform.

 # VPCs with Terraform

 [Great Documentation Explaining It All](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)

 ### creating a VPC


#### vpc.tf
 ```

# create a vpc
resource "aws_vpc" "vpc" {
    # CIDR block
    cidr_block = "${var.vpc-cidr}"
    instance_tenancy = "default"
    enable_dns_hostnames = true


    tags = {
      Name = "eng122_christian_terraform_vpc"
    }
}



# internet gateway
resource "aws_internet_gateway" "internet-gateway" {
    vpc_id = aws_vpc.vpc.id

    tags = {
        Name = "eng122_christian_terraform_ig"
    }
}

# public subnet
resource "aws_subnet" "public-subnet-1" {
    vpc_id = aws_vpc.vpc.id
    cidr_block = "${var.public-subnet-1-cidr}"
    availability_zone = "eu-west-1a"
    map_public_ip_on_launch = true

    tags = {
      Name = "eng122_christian_terraform_public_subnet_1"
    }
}

# route table
resource "aws_route_table" "public-route-table" {
    vpc_id = aws_vpc.vpc.id 

    tags = {
      Name = "eng122_christian_terraform_public_rt"
    }
  
}

# routing for route table
resource "aws_route" "default_route" {
    route_table_id = aws_route_table.public-route-table.id
    destination_cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.internet-gateway.id
}

# link the route table and the subnet together
resource "aws_route_table_association" "public-subnet-1-route-table-association" {
    subnet_id = aws_subnet.public-subnet-1.id
    route_table_id = aws_route_table.public-route-table.id
}
 ```

 #### variable.tf

 ````
 variable "vpc-cidr" {
    # cidr block
    default = "10.0.0.0/16"
    description = "VPC CIDR block"
    type = string
}

variable "public-subnet-1-cidr" {
    # public subnet's cidr
    default = "10.0.4.0/24"
    description = "public subnet 1 CIDR block"
    type = string
}
````



