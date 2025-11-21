# Terraform 모듈화

## 모듈화를 위한 폴더 정리
>home
>  >terraform_study
>  >  >main.tf
>  >  >modules
>  >  >  >web_server
>  >  >  >  >main.tf
>  >  >  >  >variables.tf
>  >  >  >  >outputs.tf

### terrafrom_study : 프로젝트 폴더
- main.tf : 대장 파일 (모듈을 불러올 역할)
- ```
      terraform {
        required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
          }
        }
      }
    provider "aws" {
      region = "ap-northeast-3"
    }
    
    resource "aws_vpc" "practice_vpc" {
      cidr_block           = "10.1.0.0/16"
      enable_dns_hostnames = true
      tags = { Name = "Practice-VPC" }
    }
    
    resource "aws_subnet" "public_subnet" {
      vpc_id                  = aws_vpc.practice_vpc.id
      cidr_block              = "10.1.1.0/24"
      map_public_ip_on_launch = true
      tags = { Name = "Public-Subnet" }
    }
    
    resource "aws_internet_gateway" "gw" {
      vpc_id = aws_vpc.practice_vpc.id
      tags = { Name = "Practice-IGW" }
    }
    
    resource "aws_route_table" "public_rt" {
      vpc_id = aws_vpc.practice_vpc.id
      route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.gw.id
      }
      tags = { Name = "Public-Route-Table" }
    }
    
    resource "aws_route_table_association" "public_assoc" {
      subnet_id      = aws_subnet.public_subnet.id
      route_table_id = aws_route_table.public_rt.id
    }
    
    data "aws_ami" "ubuntu_osaka" {
      most_recent = true
      owners      = ["099720109477"]
      filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
      }
      filter {
        name   = "virtualization-type"
        values = ["hvm"]
      }
    }
    
    module "my_web_server_1" {
      source = "./modules/web_server"  
      vpc_id      = aws_vpc.practice_vpc.id
      subnet_id   = aws_subnet.public_subnet.id
      ami_id      = data.aws_ami.ubuntu_osaka.id
      key_name    = "my-vpc-key"  
      server_name = "Module-Server-1"
    }
  ```
- 꽤나 복잡한 내용이였다...

### modules
### web_server 
- main.tf : EC2 파일
- ```
          resource "aws_security_group" "ssh_sg" {
              name        = "${var.server_name}-sg"
              description = "Allow SSH & HTTP"
              vpc_id      = var.vpc_id
            
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
             
      resource "aws_instance" "web_server" {
              ami           = var.ami_id
              instance_type = var.instance_type
              subnet_id     = var.subnet_id
              vpc_security_group_ids = [aws_security_group.ssh_sg.id]
              key_name      = var.key_name

      user_data = <<-EOF
                #!/bin/bash
                sudo apt update -y
                sudo apt install nginx -y
                sudo systemctl start nginx
              EOF
              
        tags = {
                Name = var.server_name
              }
      }
  ```
- variables.tf : 변수 파일
- ```
    variable "vpc_id" {}
    variable "subnet_id" {}
    variable "ami_id" {}
    variable "key_name" {}
    variable "server_name" { default = "My-Module-Server" }
    variable "instance_type" { default = "t3.micro" }
  ```
- outputs.tf : 출력 파일
- ```
    output "public_ip" {
      value = aws_instance.web_server.public_ip
    }
  ```

### 코드 테스트
- 중요한점은 실행 파일이 있는 terraform_study에서 실행해야함.
- `terraform init`
- `terraform apply`
- <img width="1202" height="201" alt="Image" src="https://github.com/user-attachments/assets/410f5fcf-cb48-48eb-91e6-66d14b9ab516" />
- 인스턴스가 생성되는것을 확인할 수 있다.
- 모듈화의 장점을 극대화해보자.
- 새로운 인스턴스 서버 생성하기
- <img width="456" height="284" alt="Image" src="https://github.com/user-attachments/assets/9495a6dc-3721-412a-8175-24ef66cb6c6d" />
- <img width="1213" height="122" alt="Image" src="https://github.com/user-attachments/assets/231a04c8-e1fd-48ed-be74-cc15cb491ef5" />
- 실행파일 main.tf에 코드를 추가함으로써 바로 생성이 됨을 확인했다. 
