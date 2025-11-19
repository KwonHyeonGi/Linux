# 하드코딩 탈출을 위한 main.tf 확실히 하기

## 하드코딩이란? : 코드 안에서 변경 가능한 정보를 직접 입력한 것.

### main.tf

```
# 1. Terraform 설정 블록: 사용할 프로바이더와 버전 정의
  terraform {
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0" 
      }
    }
  }
  
# 2. 프로바이더 설정: AWS에 어떤 리전(Region)을 사용할지 정의
  provider "aws" {
    region = "ap-northeast-3" 
  }
  
# 3. VPC 리소스 정의: VPC 생성
  resource "aws_vpc" "practice_vpc" {
    cidr_block = "10.1.0.0/16" 
    enable_dns_hostnames = true
    
    tags = {
      Name = "Practice-VPC-Terraform"
    }
  }
  
# 4. Public Subnet 리소스 정의: Public Subnet 생성
  resource "aws_subnet" "public_subnet" {
    vpc_id     = aws_vpc.practice_vpc.id
    cidr_block = "10.1.1.0/24"
    map_public_ip_on_launch = true
    
    tags = {
      Name = "Public-Subnet-A-Terraform"
    }
  }

# 5. Internet Gateway (IGW) 정의
  resource "aws_internet_gateway" "gw" {
    vpc_id = aws_vpc.practice_vpc.id 
  
    tags = {
      Name = "Practice-IGW"
    }
  }

# 6. Route Table 정의
    resource "aws_route_table" "public_rt" {
      vpc_id = aws_vpc.practice_vpc.id
    
      route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.gw.id
      }
    
      tags = {
        Name = "Public-Route-Table"
      }
    }

# 7. Route Table과 Public Subnet 연결 (Route Table Association)
  resource "aws_route_table_association" "public_assoc" {
    subnet_id      = aws_subnet.public_subnet.id
    route_table_id = aws_route_table.public_rt.id
  }

# 8. AWS AMI 데이터 소스 (오사카 리전의 최신 Ubuntu 22.04 LTS를 자동으로 찾습니다.)
  data "aws_ami" "ubuntu_osaka" {
    most_recent = true
    owners      = ["099720109477"] # Ubuntu 공식 계정 ID
  
    filter {
      name   = "name"
      values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
    }
  
    filter {
      name   = "virtualization-type"
      values = ["hvm"]
    }
  }

# 9. Security Group 정의 (SSH 접속 허용)
  resource "aws_security_group" "ssh_sg" {
    name        = "ssh_access_sg"
    description = "Allow SSH inbound traffic"
    vpc_id      = aws_vpc.practice_vpc.id
  
    # 인바운드 규칙: 22번 포트(SSH)를 모든 IP(0.0.0.0/0)에서 허용
    ingress {
      description = "SSH from anywhere"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    
    # 인바운드 규칙: 80번 포트(HTTP/Nginx)를 모든 IP에서 허용
    ingress {
      description = "HTTP from anywhere"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  
    # 아웃바운드 규칙: 모든 트래픽 허용 (기본값)
    egress {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
      }
    
      tags = {
        Name = "SSH-HTTP-SG"
      }
    }   
  
# 10. EC2 인스턴스 정의
  resource "aws_instance" "web_server" {
    ami           = data.aws_ami.ubuntu_osaka.id 
    instance_type = "t3.micro"
  
    # ★★★ 이전에 정의한 Subnet과 Security Group을 참조하도록 설정 ★★★
    subnet_id              = aws_subnet.public_subnet.id 
    vpc_security_group_ids = [aws_security_group.ssh_sg.id]
  
    key_name = "my-vpc-key"
  
    # EC2 시작 시 Nginx 자동 설치 User Data (EOF 오류 해결됨)
    user_data = <<-EOF
    #!/bin/bash
    sudo apt update -y
    sudo apt install nginx -y
    sudo systemctl start nginx
    EOF
    
      tags = {
        Name = "Terraform-Web-Server"
      }
    }
```
