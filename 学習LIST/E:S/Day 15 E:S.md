# E 
   
<img width="866" height="618" alt="Image" src="https://github.com/user-attachments/assets/dd7c50e4-44a4-40e5-ac8b-951a4bb7c354" />

- 실행 전 terraform plan을 통해서 수많은 에러를 확인. 
- 모든 코드 실행전 terraform plan을 통해서 에러를 확인하고, 에러코드를 확인해서 수정함으로써 실제로 오류가 나지않도록 하였음.

# E2
<img width="925" height="416" alt="Image" src="https://github.com/user-attachments/assets/2168e297-cde9-4029-8b7a-e59418e67d58" />
- 모듈화를 했을때, 폴더와 파일의 위치의 중요성을 파악하지못하였다.
- 이후 terraform_study > main.tf | modules > web_server > main.tf | variabels.tf | outputs.tf 식으로 나누었다.
- 실행 파일(main.tf)이 있는 terraform_study 폴더에서 terraform init, terraform apply 를 통해서 실행 하였음.
