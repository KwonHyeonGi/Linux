AWS 학습

1. AWS FreeTire (IAM, EC2) 기능 사용하기
2. EC2 서버 가동하여 CMD를 이용한 To-DO-List 연동해보기

[1] 
  EC2 인스턴스 서버 생성 후 .pem파일 저장(연결하는 키가 될 예정)
  downloads 에 있는 .pem 파일 실행

    ssh -i ~.pem ubuntu@ip4addr     편의상 ~(key 파일 이름), ip4addr( 인스턴스 서버 ip4주소)

  를 통해서 서버에 접속해준다.

  이 후 부터는 전에서부터 해오던
  sudo apdate
  sudo apt install nginx
  sudo systemctl start ngnix
  sudo apt insatll mysql-server
  sudo mysql_secure_insatllation -> 보안 설정
  후
  sudo mysql
    DATABASE 생성(CREATE DATABASE)
    권한 부여(GRANT ALL PRIVILEGES ON )
    TABEL 생성(CREATE TABLE)
    테이블 확인(SHOW TABLES)
      SQL 설정 완료 후 
    SQL 종료 (exit;)

  sudo apt install python3-pip python3-venv nano( 가상머신, 편집기, 수정)

  cd /var/www
  sudo mkdir new_project
  sudo chown -R ubuntu:unbuntu new_project
  cd new_project
  python3 -m venv venv
  source venv/bin/activate (venv 실행)

  후

  ip4 주소 인터넷을 통해서 입력

  welcome to nginx! 화면 확인 


  [Nginx를 통해서 연동 확인하기]

  pip install flask gunicorn mysql-connector-python 
  nano app.py 를 통해서 사용될 html 코드 입력
  python3 app.py  로 실행

  새로운 터미널에서(주의 할점 : 다시 ssh를 통해서 ec2 인스턴스 서버에 들어가야함)
  
  ssh -i ~.pem ubuntu@ip4addr
  후
  curl localhost:5000 
  사용된 코드가 나오면서 확인 완료

  [gunicorn을 통해서 연동 확인하기]

  gunicorn --bind 0.0.0.0:8000 app:app
  실행 완료 되면

  새로운 터미널에서  
  sudo nano /etc/nginx/sites-available/new_project
  sudo ln -s /etc/nginx/sites-available/new_project /etc/nginx/sites-enabled/
  sudo rm /etc/nginx/sites-enabled/default
  sudo systemctl restart nginx

  를 통해서 다시 실행 해주고

  웹사이트에 ip4주소 입력후 입력한 html 화면이 나오면 확인 완료.
  
  
-----------------------------------------------------------------------------------
Today's error

[1]  

<img width="886" height="214" alt="Image" src="https://github.com/user-attachments/assets/6960dbc1-2f3d-4780-845a-dd20dbf21861" />  

E : 새로운 터미널에서 내부테스트 할 때의 오류
S : ssh로 aws 접속 후 내부테스트

[2]   
<img width="609" height="111" alt="Image" src="https://github.com/user-attachments/assets/2f3a23f3-99b7-462f-bc95-664cf4733f00" />  

E : Nginx를 통해서 내부테스트 할 때의 오류
S : error 코드를 보면 line2 에 문제가 있는 것을 알려줌. 코드 수정 

[3]  
<img width="783" height="144" alt="Image" src="https://github.com/user-attachments/assets/792179d7-2b57-48d0-94ca-3dbe752b4c31" />  


<img width="924" height="55" alt="Image" src="https://github.com/user-attachments/assets/fa2a3a84-470b-4c22-ae65-0c1e002a6650" />  

E : Gunicorn을 통한 내부테스트 할 때의 오류
S : sudo nginx -t(nginx파일에 문제가 있는지 테스트) 후 나온 error 문구 확인후 코드 수정

