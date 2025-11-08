# 📝 ABL [Application Load Balancer] 기초 학습
## 고가용성 (High Availabliity) 와 장애복구 (Fault Tolerance) 능력

### EC2 2대 생성
> ALB-Server-A
> 서브넷 public2 사용
> > <img width="1072" height="726" alt="Image" src="https://github.com/user-attachments/assets/5b2d4a1c-4d82-442c-b46a-7cfcbdb6c847" />
> ALB-Server-B
> 서브넷 public1 사용
> > <img width="1027" height="580" alt="Image" src="https://github.com/user-attachments/assets/b14a7c08-1e7e-4902-972f-d96a9f955518" />

- 사용자 데이터(USER DATA) 설정 A/B
```
      #!/bin/bash
      sudo apt update -y
      sudo apt install nginx -y
      echo "<h1>Hello from Server A/B</h1><p>Running on Private IP: $(hostname -I)</p>" | sudo tee /var/www/html/index.nginx-debian.html
      sudo systemctl start nginx
```
-` sudo apt install nginx -y` :  인스턴스 시작과 nginx를 설치
-`echo "<h1>Hello from Server A</h1><p>Running on Private IP: $(hostname -I)</p>" | sudo tee /var/www/html/index.nginx-debian.html` : 서버 구분 페이지 HTML 출력 | 출력 내용 /var/www/html/index.nginx-debian.html으로 저장, sudo 관리자 권한으로 실행

### Security Group 설정
> ALB-EC2-SG : HTTP|포트80허용|0.0.0.0/0 으로 설정

### TG[Target Group] 생성
> 로드밸런서
> >  대상그룹
> >  WEb-Server-TG | 프로토콜 80 | VPC는 생성한 VPC로
> >  인스턴스 등록 (ALB-Server-A와 ALB-Server-B 등록)
> > <img width="1592" height="246" alt="Image" src="https://github.com/user-attachments/assets/89f5d921-d6a8-4677-a58b-5349735557e5" />

### ALB [Application Load Balancer] 구축
> 로드밸런서
> > 로드밸런서 생성
> > >  Practice-Web-ALB | 인터넷 경계(Internet-facing) | VPC 생성한 VPC 사용 | 매핑 (Public1, Public2) 모두 선택 | 보안그룹 새로 생성 ALB-SG
> > 리스너
> > >  HTTP | 포트 80 | Web_server_TG 대상그룹 지정
> > > <img width="1657" height="687" alt="Image" src="https://github.com/user-attachments/assets/8a1057bb-c8a1-4871-b939-f6beb45a813a" />
> > ALB-EC2-SG 보안그룹 업데이트
> > > http 포트 소스 VPC CIDR 로 업데이트
> > >  <img width="1657" height="687" alt="Image" src="https://github.com/user-attachments/assets/f5e54f35-d3e2-4792-9329-caac685ff38c" />

### 로드 밸런싱 정상 작동 테스트
> A 화면
> > <img width="1089" height="527" alt="Image" src="https://github.com/user-attachments/assets/4046f7f4-8a37-486e-b489-377969a9a0f7" />
> B 화면
> > <img width="869" height="282" alt="Image" src="https://github.com/user-attachments/assets/87329197-4042-4aa5-92f4-59573c2cbe97" />

### 장애 복구(HA) 테스트 
> Server-A 접속 : ssh를 이용해 ALB-Server-A 접속
> >  `sudo systemctl stop nginx` 로 ALB-Server-A 강제 중지(장애)
> > <img width="1460" height="593" alt="Image" src="https://github.com/user-attachments/assets/2c0c032f-e705-4dd5-ac7d-29b7b1b82d82" />
> > <img width="829" height="204" alt="Image" src="https://github.com/user-attachments/assets/5d6dbf1f-8919-4b7e-a5ac-f427555fd762" />
