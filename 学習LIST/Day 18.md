# Workspace, Count

## Worksapce
### 작업 공간을 분리해보기.
- main.tf
-
  ```
    provider "aws" {
      region = "ap-northeast-3"
    }
    
    resource "aws_vpc" "my_vpc" {
      cidr_block = "10.0.0.0/16"
    
      tags = {
        Name = "my-project-${terraform.workspace}"
      }
    }
    
    output "current_env" {
      value = "지금은 [ ${terraform.workspace} ] 환경입니다!"
  }
  ```
- 간단하게 vpc 만들고,
- `Name = "my-project-${terraform.workspace}" 를 통해서, 공간 이름을 변수 화 해준다.
- `terraform init`
- `terraform workspace list`
- 현재 상태를 확인해보자면, default에 있다. *로 확인 가능
- <img width="645" height="55" alt="Image" src="https://github.com/user-attachments/assets/4fe5e1d5-b9cb-475b-a28c-5175e4ccc8e5" />

### 작업공간 만들어보기
- terraform workspace new dev
- terraform workspace new prod
- dev(개발), prod(운영)라는 작업공간을 분리해봄.
- dev, prod 공간으로 이동해보기
- terraform workspace select dev
- terraform plan
- <img width="430" height="87" alt="Image" src="https://github.com/user-attachments/assets/898c5ef1-df92-4caa-a587-138b7149b125" />
- 아까 설정한 output을 통해서 확인.
- tfstate는 공간이 두개기에 두쪽 다 있는것을 확인
- <img width="650" height="86" alt="Image" src="https://github.com/user-attachments/assets/ab5a79ea-eacd-4650-9e97-af6c851a85c2" />

## Count(반복문)
### IAM(사용자)를 생성하기 위한 반복문 작성
- main.tf
- 
  ```
  provider "aws" {
      region = "ap-northeast-3"
    }
    
    resource "aws_iam_user" "students" {
      count = 5 
    
      name = "terraform-student-${count.index}"
    }
    
    output "user_names" {
      value = aws_iam_user.students[*].name
    }
  ```
- `count = 5` 를 통해서 반복해서 5번 만들어라. 라는 명령을 함
- `name = "terraform-student-${count.index}" ` : count.index가 반복할때마다 숫자가 올라가는 형태이다.
- `terraform init`
- `terraform apply`
- 직접 확인해보기
- iamge IAM
- <img width="1210" height="211" alt="Image" src="https://github.com/user-attachments/assets/42fecbbd-851e-4aeb-9566-da460e2bc180" />

### 실제 이름으로 변경해보기
- 현재 studnet0~5로 되있기에, 이걸 이름으로 변경해봄
- main.tf
  ```
    provider "aws" {
      region = "ap-northeast-3"
    }
    
    variable "user_names" {
      description = "생성할 사용자 이름 목록"
      type        = list(string)
      default     = ["kim", "lee", "park", "choi"]
    }
    
    resource "aws_iam_user" "students" {
      count = length(var.user_names)
    
      name = var.user_names[count.index]
    }
    
    output "created_users" {
      value = aws_iam_user.students[*].name
    }
  ```
- `variable "user_names"` : 명단작성할 변수
- `resource "aws"iam_user" "studnet" {` : 반복문으로 생성한다.
- `count = lenght(var.user_names)` : 명단에 있는 사람수 만큼을 length해서 반복.
- `name = var.user_names[count.index]` : 명단에서 순서대로 이름을 뽑는다. 0번째가 kim, 4번째가 choi가 될 예정
- `terraform apply`
- 새로운 모듈이 추가된것이 아니기에, init 을 통한 초기화가아닌 바로 apply
- <img width="550" height="167" alt="Image" src="https://github.com/user-attachments/assets/1d03bd24-1ecd-4b25-b833-8286fa59954f" />
- 변수와 반복문이 제대로 작동한것을 확인함.
