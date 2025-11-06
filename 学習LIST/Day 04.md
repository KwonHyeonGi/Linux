# 📝 VirtualBox를 통해서 가상환경 Ubuntu에서 실습

## 목표 : 전에 파이썬에서 만든 웹어플리케이션을 git을 통해서 연결.
### >흐름
>  >mysql 설치
>  >전용 database 생성하기
>  >Git에 있는 나의 코드를 가져오고, 가상환경(venv)에서 코드가 실행 될 수 있도록
>  >'DAY 3'에 있는 app.py를 연동
>  >내부 테스트 

* * *

### [1] MYSQL 설치

  `sudo apt install mysql-server`   
  설치 완료 후   
  `sudo mysql_secure_installation`   
  을 통해서 보안 확인(비밀번호 설정 및 root 계정에 관한 설정)   
   
  `sudo mysql`   
  을 통해서 접속해보기   
  확인 후    
  `quit;`


### [2] 전용 DATABASE 설치

  `sudo mysql`   
  을 통해서 sql 접속   
     
  `CREATE DATABASE myprojectdb;`   
     *주의* 대문자로 적어야함   
  myprojectdb라고 하는 이름으로 전용 데이터베이스 설치   

  `CREATE USER 'myprojectuser'@'localhost' IDENTIFIED BY '1234';`   
  myprojectuser라는 user는  localhost에 접속 가능하고, 비밀번호는 1234로 설정   
   
  `GRANT ALL PRIVILEGES ON myprojectdb.* TO 'myprojectuser'@'localhost';`   
  myprojectdb에 myprojectuser라는 user에게 모든 권한을 부여   
   
  `FLUSH PRIVILEGES;`   
  적용 후 새로고침   

### [3] Git에 있는 나의 코드를 가져오고, 가상환경(venv)에서 코드가 실행 될 수 있도록 하기

    `sudo apt install git python3-pip python3-venv`   
    를 통해서 venv설치 : GitHub에 있는 코드를 복제(clone)하기 위해 필요   

    `cd /var/www`   
    를 통해서 Nginx같은 웹서버가 폴더에있는 웹사이트를 보여줌   

    `sudo git clone https://github.com/KwonHyeonGi/project_life_logger.git myproject`   
    뒤에잇는 깃허브 링크를 복제(clone) 해옴   

    `sudo chown -R hyoengi:hyeongi myproject`   
    sudo 로 폴더를 만들고 있기에, 폴더 주인이 root로 되어있음.   
    그렇기에 계정을 root로 바꿔야함    

    `cd project`   
    project 폴더로 이동 후   

    `python3 -m venv venv`   
    venv 라는 이름으로 가상환경폴더를 생성   

    `source venv/bin/activate`   
    가상환경 활성화   

    `pip install -r requirements.txt`   
    를 통해서 flask, gunicorn 등등을 설치   
    나는 없어서 requirements 파일이 없어서    
    `pip install flask gunicorn mysql-connector-python`   
    을 통해서 직접 설치 함   

### [4] 'DAY 3'에 있는 app.py를 연동      

  우선 가져온 'LIFE LOGGER'의 코드가 virtualbox의 mysql에 접속 할 수 있도록 수정이 필요   

  `sudo apt insatll nano`   
  서버에서 코드를 수정해주는 편집기 nano 설치   

  LIFE LOOGER에 있는 app.py의 정보 변경을 위하여   
  myproject/'DAY 3' 폴더로 이동   
  `cd 'DAY 3`   
  후   

  `nano app.py`   
   app.py 내용이 쭉 나옴   
   그 중    
   ```
      'user': 'myprojectuser',      # Day 3에서 만든 아이디
      'password': '1234',           # Day 3에서 설정한 비번
      'host': 'localhost',        # '127.0.0.1' 또는 'localhost'
      'database': 'myprojectdb'     # Day 3에서 만든 DB 이름
  ```

    이런 식으로 수정을 해줌   

    완료 후   


### [5]. 내부 테스트 
    `python3 app.py`   
    를 통해서 서버를 열어줌   

    Running on http://~~~ 라고 뜨면 성공   

    새로운 터미널창을 열어서    
    `curl localhost:5000`   
    로 연결 확인   

```   
*** 중간에 있었던 오류 *****   
내부 테스트 중   

Acces denied for user 'myprojectuser'@'localhost' to database 'myprojectydb' hyeongi@hyoengi-virtualBox:var/www/project$ 라고 나옴   

: app.py 수정 할 때 database 의 이름에 오타가 있었음.   
다시 nano app.py로 수정 해줌
```
