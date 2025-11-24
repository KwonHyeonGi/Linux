# Terraform Registry
## Registrty를 통해서 main.tf 코드를 줋이고, module을 불러오는 매우 편리한 기능

### main tf
- 저번에 했던 폴더가 아닌 새로운 폴더에서 main.tf 작성
- ```
    provider "aws" {
        region = "ap-northeast-3"
      }
      
      module "vpc" {
        source = "terraform-aws-modules/vpc/aws"
      
        name = "my-registry-vpc"
        cidr = "10.0.0.0/16"
      
        azs             = ["ap-northeast-3a", "ap-northeast-3c"]
        private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
        public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
      
        enable_nat_gateway = false 
        enable_vpn_gateway = false
      
        tags = {
          Terraform = "true"
          Environment = "dev"
        }
      }
  ```
- 원래 작성했던 main.tf와 다르게 매우 간단해짐
- 생략된게 있다면 `terraform { required_providers { ~~ `인데 이거는 버전 고정을 위해서 작성해야하지만, 이번 실습에서는 최대한 짧게 코드를 사용하고자 생략을 함. ( 생략 시 가장 최신 버전 사용 )
- `source = "terraform-aws-moudles/vpc/aws" ` : 출처는 terraform-aws-modules이며 그 사람의 vpc를 가져와라
- `name`, `cidr` : 기본 설정
- `azs`, `private_subnets`, `public_subnets`  : 가용 영역과 서브넷
- `enable_nat_gateway`, `enable_vpn_gateway` : 옵션 설정 / false를 한 이유는 실습이기에...
- `tags` : 꼬리표

### 🧧원래 사용했던 main.tf와 다른점🧧
- VPC : `resource "aws_vpc"~~ `| Registry 모듈을 통한 자동생성
- Subnet : `resource "aws_subnet" ` | `public_subnets = [] `
- IGW : `resource "aws_internet_gateway" ` | Registry 모듈을 통한 자동생성
- Routetable : `resource "aws_route_table" ` | Regisrty 모듈을 통한 자동생성 및 자동 연결

### init
- terraform init
- <img width="750" height="261" alt="Image" src="https://github.com/user-attachments/assets/262f51ec-98e4-4ae5-807b-3ab1d95f96ca" />
- 파일 / 폴더가 생성됨이 확인

### 불러온 폴더 및 파일
>day16_registry
>  >.terraform
>  > main.tf
>  > terraform.tfstate
>  >  > modules
>  >  > providers
>  >  >  > moudles.json
>  >  >  > vpc
>  >  >  >   > variables.tf
>  >  >  >   > vpc-flow-logs.tf
>  >  >  >   > maint.tf
>  >  >  >   > outputs.tf
>  >  >  >   > version.tf

- 즉 modules 안에 지금 당장 필요한 main.tf | variables.tf | outputs.tf 가 있음을  확인

### terraform apply
- 내부에 모듈도 확인했기에 apply 해봄
- 이를 통해서 VPC(가상 네트워크) 생성 완료.

### 인스턴스 생성에 관한 나의 생각
- VPC를 생성했기에 EC2 인스턴스도 생성해보고 싶다는 생각이 들어 코드를 확인해봄
- ```
    module "ec2_instance" {
         source  = "terraform-aws-modules/ec2-instance/aws"
      
        name = "Registry-Server"
      
        instance_type          = "t3.micro"
        key_name               = "my-vpc-key"  
        monitoring             = true
        
        vpc_security_group_ids = [module.vpc.default_security_group_id] 
        
        subnet_id              = module.vpc.public_subnets[0]             
      
        tags = {
          Terraform   = "true"
          Environment = "dev"
        }
      }
  ```
  - 저번에 생성한 코드보다 더 길어졌다. 저번에는 modules 안에 variables.tf를 통해서 변수설정을 해줬기에 간단하게 생성 할 수 있었지만. 이번에는 가져온 파일이기에 변수설정이 안되있음.
  - 실제로 `vpc_security_group_ids` 부분은 위에서 생성한 보안그룹을 사용하게 하거나
  - subnet 또한 위에서 생선한 subnet을 그대로 가져오게 한다.
  - 그렇다면 variables.tf를 수정해서 변수설정을 한다면 main.tf에서의 서버생성 코드가 더 간단해질 수 있는거 아닌가?
  - 의 질문에서는, terraform-aws-modules/vpc/aws 같이 레지스트리 모듈을 다운로드한경우에는 수정이 불가능하다(폴더가 숨겨져있기에)
  - 그래서 오늘은 vpc생성까지로 마무리.
  
