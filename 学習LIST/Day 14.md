# Terraform.tfstate 파일 

## Terraform.tfstate파일 이해하기
- Day 13에서의 리소스파일에 변수설정 후 생성된 파일 Terraform.tfstate
- Terraform.tfstate : 기억하는 장소? / 매우 중요한 장소이기에 수정, 삭제시 복구가 매우 힘듬.
- terraform.tfstate 내용 확인해보기 / 특히 IP주소 찾아보기
- `cat terraform.tfstate`
- <img width="1118" height="760" alt="Image" src="https://github.com/user-attachments/assets/668d967f-560e-4e39-9b42-b22b9d1a9288" />
- 너무 방대한 내용이나와 찾기가 힘듬.
- `cat terraform.tfstate | grep "public_ip"
- <img width="652" height="112" alt="Image" src="https://github.com/user-attachments/assets/aad069a3-d736-4682-a35b-868032ea650d" />
- 검색을 통해 원하는 부분만 확인가능.

## Drift 만들기
### AWS콘솔창에서 생성한 인스턴스 삭제 후, terraform 다시 실행해보기
- AWS콘솔창에서 직접 삭제시, 확인전까지는 tfstate는 서버가 생성되어있는것으로 인식 하고 있을것이라 생각.
- 그 후 `terraform apply` 를 하면 tfstate는 이미 서버가 있기에, 만들지않을지. 확인 후 서버가 삭제된것을 확인하고 다시 서버를 설치할지 확인.
- <img width="1254" height="444" alt="Image" src="https://github.com/user-attachments/assets/7501f254-b272-45d6-878d-3a241b776cd8" />
- 우선 인스턴스(서버) 삭제
- `terrafrom plan` 을 통해서 변경사항 확인
- <img width="702" height="130" alt="Image" src="https://github.com/user-attachments/assets/02d6a722-d9c9-40a0-809b-fc115dc234bd" />
- terraform.tfstate의 자가 치유 : 서버가 삭제됨을 확인하고, main.tf(리소스파일) 대로 다시 서버를 생성 하였음. (1 to add)
- <img width="610" height="125" alt="Image" src="https://github.com/user-attachments/assets/3a17f45f-2906-4954-86c9-4bbae6093882" />
- 전에 생성했던 IP와 URL과는 다른것을 볼 수 있음.


#### Q. 서버 IP, URL이 바뀌었다는것은 무슨 문제가 생겨서 서버를 재생성 시 IP가 바뀌기에 유동적이지않음. 불편함을 겪는게 아닌지?
- <img width="721" height="581" alt="Image" src="https://github.com/user-attachments/assets/95a275bc-1b16-4f02-9f5b-cd038c554e1b" />
- 실무에서는 고정 IP를 사용한다고 한다..
