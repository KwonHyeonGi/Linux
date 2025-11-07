# error 1
## ssh 접속 오류.
<img width="707" height="72" alt="Image" src="https://github.com/user-attachments/assets/7e45963d-2803-43d2-8eb5-0446ede3b07a" />   

## Solution
###  subnet 설정 시 Private가 아닌 Public으로 되어있었음.
<img width="1657" height="461" alt="Image" src="https://github.com/user-attachments/assets/2272e733-8d17-4cce-a62d-5570e4b2671a" />

# error 2
## Gunicorn 접속 오류.
<img width="1096" height="477" alt="Image" src="https://github.com/user-attachments/assets/79c0566f-48b5-47b7-a2d2-d83fa2590cbe" />   

# Solution 
## File "/var/www/new_project?app.py", line 84 부분의 오류가 있어 수정 
<img width="1095" height="349" alt="Image" src="https://github.com/user-attachments/assets/395abc7b-b841-411b-b228-b0351e9cc533" />

# error 3
## Nginx 통신 문제
<img width="1018" height="111" alt="Image" src="https://github.com/user-attachments/assets/b46162fe-17f2-4090-9a29-899efa33a50f" />

# Solution
## `sudo nginx -t` 를 통해서 미리 실행을 해보고, 오류가 있는 부분 확인 etc/nginx/sites-enabled/new_project 에서 sock 포트로 변경하며 오류가 생김. 수정.
<img width="705" height="284" alt="Image" src="https://github.com/user-attachments/assets/262f77fd-a5ad-4baa-bb64-f87f6b844106" />
