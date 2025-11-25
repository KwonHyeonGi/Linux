#S3
## Simple Storage Service : 클라우드 외장하드 라고 생각하면 편하다.

### terraform을 사용하면서, terraform.tfstate를 보존하기 위해서, S3에 저장되게 할 예정.
-main.tf부터 시작한다.
- ```
        provider "aws" {
        region = "ap-northeast-3"
      }
      
      resource "aws_s3_bucket" "mys3" {
        bucket = "hyeongi-tf-state-bucket-test" 
      }
  
      resource "aws_s3_bucket_versioning" "mys3_versioning" {
        bucket = aws_s3_bucket.mys3.id
        versioning_configuration {
          status = "Enabled"
        }
      } 
  ```
- `resource "aws_s3_bucket" "mys3"` 를 통해서, S3를 제작해준다
- `resource "aws_s3_bucket_versioning" "mys3_versioning"`을 통해서, 안전장치를 더해준다(파일을 실수로 덮어써도, 버전으로 저장되기에 전에 버전으로 되돌리기가 가능해짐)
- `terraform init`
- `terraform apply`
- <img width="928" height="205" alt="Image" src="https://github.com/user-attachments/assets/9e64221e-9afc-4acf-b25c-8c8f758e6f8e" />

### Backend 설정
- 연결을 통해서, S3에 저장되게 설정
- main.tf 아까 입력한 S3 윗부분에 적을 예정
- 
  ```
      terraform {
      required_providers {
        aws = {
          source  = "hashicorp/aws"
          version = "~> 5.0"
        }
      }
    
      backend "s3" {
        bucket = "hyeongi-tf-state-bucket-test"  #아까 생성한 bukcet 이름으로
        key    = "terraform.tfstate"       
        region = "ap-northeast-3"       
      }
    }
    
    provider "aws" {
      region = "ap-northeast-3"
    }
    
  ```
- `terraform init`
- 여기서 실수가 한번 있었다. 처음 설정할때, version을 정해주지않아, 최신버전으로 입력됬지만, 이번 main.tf에는 version 5.0으로 설정했기에 맞지않음.
- `terraform init -upgrade`
- <img width="931" height="315" alt="Image" src="https://github.com/user-attachments/assets/2f9d6543-d5b8-4032-be31-4b6dc08e8a81" />
- S3에 복사가 된 모습을 확인할 수 있음.

### DynamoDB : 잠금장치
- 누군가와 동시에 사용, terraform apply가 됬을 경우 문제가 발생할 수 있기에, 한명이 사용중이면 다른 사람은 사용이 안되게 해보자
- `nano main.tf`
-
  ```
  resource "aws_dynamodb_table" "terraform_locks" {
      name         = "hyeongi-tf-locks" 
      billing_mode = "PAY_PER_REQUEST"  
      hash_key     = "LockID"            
    
      attribute {
        name = "LockID"
        type = "S"
      }
    }
  ```
- `name` : 잠금 장치 이름, 원하는 대로
- `billing_mode` : 금액에 관한건데, 사용할때만 지출하기로 pay_per_request
- 저장 후,
- `terraform apply`
- DynamoDB 테이블 생성 완료.
- 이제 생성한 DynamoDB를 잠금 장치로 사용하기 위한 코드 작성
- 다시 `nano main.tf`
- 
```
     terraform {
      # ... (생략) ...
      backend "s3" {
        bucket         = hyeongi-tf-state-bucket-test
        key            = "terraform.tfstate"
        region         = "ap-northeast-3"
        
        dynamodb_table = "hyeongi-tf-locks" 
      }
    }
```
- 맨 backend안에 `dynamodb_table = "hyeogi-tf-locks"`를 추가 해줌.
- 저장 후, 백엔드 설정이 변경되었기 때문에 초기화
- `terraform init -reconfigure`
- <img width="865" height="231" alt="Image" src="https://github.com/user-attachments/assets/56392da5-7b83-453b-94d2-dfc7d81cd4c2" />
- 리소스로 locks라고 뜨고 있음을 확인!!
  
