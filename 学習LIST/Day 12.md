# Terraform VPC/Subnet  확장 및 배포

## main.tf
- 전에 사용한 `main.tf`
- ```
  # 1. Terraform 설정 블록: 사용할 프로바이더와 버전 정의
    terraform {
      required_providers {
        aws = {
          # AWS 프로바이더를 사용하겠다고 선언
          source  = "hashicorp/aws"
          # AWS 프로바이더의 버전 지정
          version = "~> 5.0" 
        }
      }
    }
    
    # 2. 프로바이더 설정: AWS에 어떤 리전(Region)을 사용할지 정의
    provider "aws" {
      # 도쿄 리전(ap-northeast-1) 대신, 주인님께서 사용하신 오사카 리전(ap-northeast-3)을 사용합니다.
      region = "ap-northeast-3" 
    }
    
    # 3. VPC 리소스 정의: VPC 생성
    resource "aws_vpc" "practice_vpc" {
      # VPC의 IP 주소 범위 (CIDR)를 지정
      cidr_block       = "10.1.0.0/16" 
      # DNS 호스트 이름 지원 활성화
      enable_dns_hostnames = true
      
      # AWS 콘솔에 표시될 태그 (이름) 정의
      tags = {
        Name = "Practice-VPC-Terraform"
      }
    }
    
    # 4. Public Subnet 리소스 정의: Public Subnet 생성
    resource "aws_subnet" "public_subnet" {
      # 이 Subnet을 연결할 VPC 지정
      vpc_id     = aws_vpc.practice_vpc.id
      # Subnet의 CIDR 블록 지정 (VPC 범위 내에서 쪼개어 사용)
      cidr_block = "10.1.1.0/24"
      # Public Subnet이므로 인스턴스 시작 시 Public IP를 자동 할당
      map_public_ip_on_launch = true
      
      tags = {
        Name = "Public-Subnet-A-Terraform"
      }
    }

    # 추가 내용
    # 5. Internet Gateway (IGW) 정의
  resource "aws_internet_gateway" "gw" {
    # IGW를 연결할 VPC 지정
    vpc_id = aws_vpc.practice_vpc.id 
  
    tags = {
      Name = "Practice-IGW"
    }
  }

    # 6. Route Table 정의
  resource "aws_route_table" "public_rt" {
    vpc_id = aws_vpc.practice_vpc.id
  
    # 0.0.0.0/0 (모든 외부 트래픽)을 IGW로 보내도록 설정 (Public Subnet의 특징)
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
  ```

- `terraform plan`을 통한 코드 추가 확인
- `terraform apply --auto-approve` 로 코드를 AWS에 적용
- 할 예정이였으나. 오류가 너무 많다. 코드문제와 기본 인스턴스 설치 했을때의 오류가 너무 많아, 내일 다시 도전해볼 예정.

## 내가 확인한 문제 
> 기본 VPC를 사용하고 sudo apt update 및 install을 할때 제대로 되지 않음
> cli를 통해서 vpc랑 추가할 예정인데, 기본 vpc로 설치가 안되서 vpc를 만들어서 해봄.
> 그래서 그런건지 생각보다 오류가 생긴게 아닌가? 라고 생각이 듦. 기본적인 main.tf의 오류도 있는거라고 생각함.


# Terraform VPC/Subnet  확장 및 배포
## 조금 쉬다와서 다시 함으로써 성공..

### Terraform, cli를 사용하여 aws의 시스템을 설정 할 예정.
- 전에 했던, terraform에서 이어질 예정
- 전에 사용하던 main.tf에서 추가 될 코드 확인
- ```
  # 추가 내용
  # 5. Internet Gateway (IGW) 정의
  resource "aws_internet_gateway" "gw" {
  # IGW를 연결할 VPC 지정
  vpc_id = aws_vpc.practice_vpc.id
   tags = {
    Name = "Practice-IGW"
    }
  }
    
    # 6. Route Table 정의
    resource "aws_route_table" "public_rt" {
    vpc_id = aws_vpc.practice_vpc.id
    # 0.0.0.0/0 (모든 외부 트래픽)을 IGW로 보내도록 설정 (Public Subnet의 특징)
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
    subnet_id = aws_subnet.public_subnet.id 
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
      instance_type = "t2.micro"
    
      # ★★★ 이전에 정의한 Subnet과 Security Group을 참조하도록 설정 ★★★
      subnet_id              = aws_subnet.public_subnet.id 
      vpc_security_group_ids = [aws_security_group.ssh_sg.id]
    
      # SSH 접속을 위한 Key Pair 이름 지정 (사용자 계정에 등록된 키 이름으로 변경하세요!)
      key_name = "my-vpc.key"
    
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
- <img width="744" height="228" alt="Image" src="https://github.com/user-attachments/assets/deb066e6-f697-4c97-a1b7-a9836792cdcb" />
- 확인 완료
- <img width="1597" height="675" alt="Image" src="https://github.com/user-attachments/assets/38093b9d-2fea-43bf-83f4-964b53e69de3" />
- 인스턴스 설치 확인 완료
- <img width="1061" height="320" alt="Image" src="https://github.com/user-attachments/assets/0668390a-8a21-45d3-979e-bae926b106dc" />
- Nginx 웹 서버 테스트 완료





* * *
- 솔직히 오늘한게 좀 이해가 안되서 내일 다시 봐야할듯.
  
  
