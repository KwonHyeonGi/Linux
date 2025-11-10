# Gunicorn의 소켓통신

## [1] Systemd에 Gunicorn의 설명서 파일 작성
  -`sudo nano /etc/systemd/system/gunicorn.service`
  -`service부분에서 내 저장공간에 맞게 수정`
  ```
              [Unit]
        Description=Gunicorn instance to serve my-todo-app
        After=network.target
        
        [Service]
        User=ubuntu
        Group=www-data
        WorkingDirectory=/var/www/new_project
        Environment="PATH=/var/www/new_project/venv/bin"
        ExecStart=/var/www/new_project/venv/bin/gunicorn --workers 3 --bind unix:gunicorn.sock -m 007 app:app
        
        [Install]
        WantedBy=multi-user.target
  ```

## > [2] Systmed 서비스 활성화
>    > `새 설명서 인식시키기`
>    >   > `sudo systemctl daemon-reload`
>    > `서비스 시작`
>    >   > `sudo systemctl start gunicorn`
>    >   `서비스 상태 확인`
>    >   >  `sudo systemctl status gunicorn`
>    >   ` 서비스 자동 실행`
>    >   > `sudo systemctl enable gunicorn`


## [3] Nginx와 Gunicorn 통신
### 이 전에는 8000번 포트에서 Nginx와 Gunicorn이 통신 했지만, 이번에는 Systemd의 소켓파일(.sock)로 통신할 예정

  - `sudo nano /etc/nginx/sites-available/new_project`   를 통해서 포트 방식에서 소켓방식으로 변경해준다.   
  - `proxy_pass http://127.0.0.1:8000;` -> `proxy_pass http://unix:/var/www/new_project/gunicorn.sock;`   변경 완료
  - `sudo systemctl restart nginx` : Nginx 재시작

오늘의 학습 결과 : Gunicorn을 수동으로 8000포트로 켜지않아도 자동으로 연결되는 systemd 서비스를 생성해, 자동화를 학습함.  
