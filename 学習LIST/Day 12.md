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
