AWS

WEBSERVER 와 DATABASE를 분리해서 저장하기.  
VPC(Virtual Private Cloud)라는 가상 네트워크를 이용할 예정.

[1]    

  **VPC를 생선한다.**    
  
  1. Public subnet  ->  공인 ip를 획득할 수 있게, IPv4 자동할당  
  2. Private subnet  -> DATABASE가 저장 될 공간  

[2]  

  **SUBNET GROUP 설정을 해준다.**  
  
  Subnet Group 설정을 통해서 RDS(사용할 database)를 어디에 저장할건지 설정이 가능함.  

[3]  

  **RDS에서 데이터 생성**  
    
  생성되는 위치는 아까 생성한 VPC 네트워크 환경.  
  설정한 Subnet Group의 private 서브넷을 선택.  
  VPC 보안 그룹은 새로 생성해준다.   

[4]
  **Public 서버 생성**  
  
  EC2가상서버 생성하기.  
  
  EC2 인스턴스 생성.  
  생성되는 위치는 아까 생성한 VPC 네트워크 환경.  
  Subnet설정은 아까 생성한 Subnet Group에서의 public.  
  보안그룹 생성.  
  인바운드 규칙 ssh, http 0.0.0.0/0 으로 : 어디서든 들어올 수 있게 해준다는 뜻.  
  
[5]
  **RDS(데이터)가 있는 Private 구역에 EC2그룹에서만 접속 가능하게 설정**  
  
  EC2에서 제작한 보안그룹의 ID를   
  RDS생성시 만든 보안그룹의 인바운드 규칙으로 넣어준다.  이 보안그룹이 (EC2에서 만든 보안그룹) 속한 서버에서만 Private 구역에 접속이 가능하다.   
  라는 것을 설정 해주는 것임.

  
  설정 완료 한 후.

  Ngnix와 gunicorn 을 통한 내부 테스트.


 <img width="768" height="91" alt="Image" src="https://github.com/user-attachments/assets/a8d304e3-1f74-4289-9139-fc776857bd80" />  
 
  완료.
