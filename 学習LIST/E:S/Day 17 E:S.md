# terraform init에서의 오류
- <img width="1109" height="325" alt="Image" src="https://github.com/user-attachments/assets/e3ddde86-ac85-4aaf-af09-a81bf2343406" />
- main.tf 작성 중, 처음에 init, apply했을떄는 version 확립을 안해줬다.
  ```
    provider "aws" {
      region = "ap-northeast-3"
    }

    resource ~~~~~~~~~~~~~~~~~~~
  ```
- 하지만 main.tf를 수정 하면서 버전을 확립하니.
  ```
    terraform {
      required_providers {
        aws = {
          source = "hashicorp/aws"
          version = "~> 5.0"
        }
      }

    backend ~~~~~~~~~~~~~~~~~~~
  ```
- 저런식으로 오류가 발생했다.
- 즉, required_provider로 확립을 안해주면, 제일 최신 버전을 사용함 (6.22.1), 하지만 우리는 5.0을 쓰라고 함. 안맞음. 충돌
- `terraform init -upgrade` : 기존에 잠겨있던 버전을 무시하고, 지금의 코드(main.tf)에 맞는 버전으로 다시 다운로드 해라. 라는 코드
