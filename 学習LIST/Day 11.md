# S3, ACL
## S3 : 외부의 파일을 저장하는 공간, Access Key를 이용하기엔 위험부담이 있기에, IAM Role을 이용하여 Access Key가 없어도 특정한 역할을 이용 가능하게 해보기.

### S3 버킷 생성
- S3 버킷 생성. 버킷 이름은 my-infra-log-bucket-hyeongi
- <img width="1195" height="644" alt="Image" src="https://github.com/user-attachments/assets/67854ecb-4f99-4364-a4af-3b170a5fc2c4" />

* * *

### IAM 정책 생성
- IAM 서비스의 정책 생성
- JSON 탭에서 파일 업로드, 파일 다운로드, 파일리스트 허용하는 최소 권한 정책 생성
  ```
    JSON
    {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "AllowS3AccessForEC2",
                  "Effect": "Allow",
                  "Action": [
                      "s3:PutObject",
                      "s3:GetObject"
                  ],
                  "Resource": "arn:aws:s3:::my-infra-log-bucket-hyeongi/*"
              },
              {
                  "Sid": "AllowListBucket",
                  "Effect": "Allow",
                  "Action": "s3:ListBucket",
                  "Resource": "arn:aws:s3:::my-infra-log-bucket-hyeongi"
              }
          ]
      }
  ```
- 정책 생성, 이름은 S3-PutGet-Minimal-Policy
- <img width="1150" height="234" alt="Image" src="https://github.com/user-attachments/assets/4dd6c0c3-5a85-4381-835c-838fa4fe5f96" />

* * *

### IAM 역할 생성
- 권한 정책 연결을 아까 생성한 'S3-PutGet-Minimal-Policy' 로 선택 후 생성
- 이름은 EC2-S3-Uploader-Role
- <img width="1150" height="470" alt="Image" src="https://github.com/user-attachments/assets/7c1ebce5-8022-4867-9868-9bd3ecbb6f18" />

### AWS CLI 환경 확인
- `aws --version`을 통해 확인
- <img width="490" height="55" alt="Image" src="https://github.com/user-attachments/assets/f5d815e4-ad52-48ab-a523-8c4b7e3b5a99" />
- aws cli가 없음으로 재설치(저번 시간 했기에, 빠르게 넘어감)
- 
  ```
    sudo apt update
    sudo apt install curl unzip -y

    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
  ```
- `aws --version`
- <img width="686" height="42" alt="Image" src="https://github.com/user-attachments/assets/d3ecd852-c3f3-4ffa-9ba0-fa2a89eacf1a" />

* * *

### EC2에 Role 연결 및 최종 검증
- 인스턴스 -> 작업 -> 보안 -> IAM 역할 수정 : 생성한 EC2-S3-Uploader-Role 연결
- `aws s3 cp secured_log.txt s3://my-infra-log-bucket-hyeongi`
- 업로드 된것을 확인
- <img width="769" height="65" alt="Image" src="https://github.com/user-attachments/assets/7d91d012-5f3e-493d-9917-24ab0e753c60" />

####
Access Key를 사용하면 위험부담이 있기에, 특정한 역할(업로드, 다운로드)는 Access Key가 없어도 사용할 수 있게하는 역할 부여.   
Access Key를 사용하지않고, 임시 자격을 부여했기에 권한유출위험이 덜함.

* * *

## ACL : Network ACL, 외부 경계 방화벽
### 네트워크 ACL 개념 이해
- ACL 규칙편집
- <img width="1585" height="567" alt="Image" src="https://github.com/user-attachments/assets/99b620b3-50e8-4583-8749-49e75d49cabd" />
- 규칙 90 : ssh | 내 공인 IP | Deny
- 규칙 100 : ssh | 모든 IP | Allow
- <img width="689" height="98" alt="Image" src="https://github.com/user-attachments/assets/76fc986e-42b4-49dc-86d8-49a81752db90" />
- 우선 ACL은 SG보다 먼저 작용함을 알 수 있음 : SG는 ssh 모두 허용상태이지만, 내 IP가 접속하는것이 거부됬기에.
- ALC은 규칙번호 순서대로 작동한다. 90번 규칙을 통해 나의 IP를 거부하고, 100번 규칙을 통해 모든 IP를 허용했지만, 내 IP가 거부됨을 확인.
