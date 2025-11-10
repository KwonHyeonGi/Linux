# Shell Script 심화
## 흐름
>서버 초기설정(필수 패키지 설치 및 방화벽 설정) 스크립트 작성
>  >에러 발생시 바로 확인가능한 타임스탬프 추가, pipefail을 통한 에러 발생시 불가피한 사항 예방.
>사용자 생성 및 권한 부여

### 서버 초기설정 스크립트 ( setup_server.sh )
```
        #!/bin/bash
        # ----------------------------------------------------------------------
        # 1. 스크립트 실행 규칙 정의 및 로깅 설정 (실무 필수)
        # ----------------------------------------------------------------------
        set -euo pipefail  # 에러 발생 시 즉시 종료 (Fail-fast)
        
        # 로그 파일 경로 정의
        LOG_FILE="./setup_server_$(date +%Y%m%d_%H%M%S).log"
        
        # 표준 출력(1)과 에러 출력(2)을 로그 파일과 화면에 동시에 기록
        # 이 코드는 스크립트 내부에서만 사용해야 합니다.
        exec > >(tee -a ${LOG_FILE}) 2>&1 
        
        log_message() {
            echo "$(date '+%Y-%m-%d %H:%M:%S') [INFO] $1"
        }
        log_message "Server setup script started."
        
        # ----------------------------------------------------------------------
        # 2. 시스템 업데이트 및 필수 패키지 설치
        # ----------------------------------------------------------------------
        log_message "System update started."
        
        sudo apt update -y
        
        log_message "Installing essential packages: Nginx, Python3, Git."
        
        # Nginx와 Python을 설치합니다.
        sudo apt install -y nginx python3 python3-venv git
        
        # Nginx 서비스 시작
        log_message "Starting and enabling Nginx service."
        
        sudo systemctl start nginx
        
        sudo systemctl enable nginx
        
        # ----------------------------------------------------------------------
        # 3. 방화벽 (UFW) 설정 추가 (보안 강화)
        # ----------------------------------------------------------------------
        log_message "Setting up firewall (UFW)."
        
        sudo ufw default deny incoming
        
        sudo ufw default allow outgoing
        
        sudo ufw allow 22/tcp  # SSH
        
        sudo ufw allow 80/tcp  # HTTP (Nginx)
        
        sudo ufw --force enable
        
        log_message "UFW enabled. Only 22 and 80 ports are allowed."
        
        # ----------------------------------------------------------------------
        # 4. 스크립트 최종 종료 (정상 종료)
        # ----------------------------------------------------------------------
        log_message "SUCCESS: Server setup completed successfully."
        
        exit 0
```

- `set -euo pipefail` : pipefail에서 1이 나오면 -euo를 통해서 즉시 종료 시키는 명령어 (pipefail은 코드에서 에러가 나면 1, 에러가 없으면 0으로 반환해줌)

- `LOG_FILE="./setup_server_$(date +%Y%m%d_%H%M%S).log"` : 실행하면서 나오는 로그 파일의 경로설정, 나오는 로그에 전부 타임스탬프 적용.

- ` exec > >(tee -a ${LOG_FILE}) 2>&1 ` : exec: 모든 출력은 이제부터 이런식으로 된다: | > > : 에러출력, 표준출력 | ( ) : 괄호 안에있는 프로세스에 나올 예정 | (tee -a${LOG_FILE}) : 출력되는 로그를 저장/출력을 한번에 해준다. -a : append / ${LOG_FILE} : log파일의 역할(변수) | 2>&1 : 2(표준에러)를 1(표준출력)의 방향과 동일하게 변경

#2, #3을 통한 필수패키지 설치 및 방화벽 설정

스크립트 저장 후 활성화

- `./setup_server.sh`  
- `ls` 를 통해서 홈 디렉토리에 setup_server_simple.log 파일 있는거 확인
- `echo $?` 을 통해서 총 에러가 몇번 났는지 확인 | 결과값 : 0 | 에러가 없다.
- 스크립트 문제 없음.


### 직접 에러 발생시키기

- pipefail을 통한 에러 발생시 상황 연출
- `nano setup_server.sh`
  
```
      # ... (Nginx 서비스 시작 완료 부분)
      sudo systemctl enable nginx
      
      # ----------------------------------------------------------------------
      # ✨ 3. 안정성 검증을 위한 트러블 유발 로직 추가 (임시)
      # ----------------------------------------------------------------------
      log_message "Simulating critical failure using 'false' command for set -e test."
      false 
      
      # ----------------------------------------------------------------------
      # 4. 방화벽 (UFW) 설정 추가 (보안 강화)
      # ----------------------------------------------------------------------
      log_message "Setting up firewall (UFW)." # ✨ 이 메시지는 출력되면 안됩니다!

```
-`false` : 에러
- 뒤의 `log_message "setting up firewall (UFW)."` 를 주석처리한 이유는 조금 더 확실하게 확인하기 위함.
- `./setup_server.sh`
- 에러 발생 후 ssh전체가 종료 됨. pipefail이 잘 작동됨을 확인.
<img width="984" height="138" alt="Image" src="https://github.com/user-attachments/assets/37a292df-42cb-40e8-b820-56f4dc6e6839" />   



- false 삭제

### 운영사용자 생성 

```
      # ----------------------------------------------------------------------
      # ✨ 3. 운영 사용자 생성 및 권한 설정 (보안 강화)
      # ----------------------------------------------------------------------
      USERNAME="ops_user"
      
      log_message "Creating new non-root user: ${USERNAME}"
      
      if id "${USERNAME}" &>/dev/null; then
          log_message "User ${USERNAME} already exists. Skipping creation."
      else
          # -m: 홈 디렉토리 생성, -s /bin/bash: 기본 쉘 설정
          sudo useradd -m -s /bin/bash "${USERNAME}" 
          # NOPASSWD:ALL을 설정하여 ops_user가 비밀번호 없이 sudo 명령을 실행하도록 허용
          echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/"${USERNAME}"
          log_message "User ${USERNAME} created and given NOPASSWD sudo access."
      fi
      
      # ----------------------------------------------------------------------
      # 4. 방화벽 (UFW) 설정 추가 (보안 강화)
      # ----------------------------------------------------------------------
      log_message "Setting up firewall (UFW)."
      # ... (기존 UFW 코드는 그대로 둡니다.)
```

- `if id "${USERNAME}" &>/dev/null; then log_message "User ${USERNAME} already exists. Skipping creation."` : username이 있던, 없던 출력값은 버린다.(불필요한 메세지 삭제)0
- `if id "${USERNAME}" &>/dev/null` : ${USERNAME}의 ID가 존재하는지 확인(중복 생성 예방)
  
-  `else
          sudo useradd -m -s /bin/bash "${USERNAME}" 
          echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/"${USERNAME}"
          log_message "User ${USERNAME} created and given NOPASSWD sudo access." `
-  `useradd` : 새 사용자 추가
- `-m` : 사용자의 home directory 지정 /home/ops_user
- `-s /bin/bash` : 기본 쉘을 지정(ssh로 접속했을 때 명령어를 실행할 수 있는 BASH쉘 제공
- `echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL"` : echo를 통한 출력 ops_user가 모든터미널(ALL)에서 NOPASSWD로 모든 관리자 명령을 실행할 수 있게 하는 규칙 설정
- `sudo tee ect/sudoers.d/"${USERNAME}"` : sudo 권한 설정 파일을 모아두는 시스템 디렉토리(.d 이기에)에 ${USERNAME}을 파일 저장/생성을한다. 일반사용자 권한으로는 불가능 하기에 sudo를 통해서 가능하게 해줌
- 즉 ` echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/"${USERNAME}"` 을 A|sudo tee /etc/sudoers.d/"${USERNAME}" 로 표현하면 A의 규칙을 tee 프로세스로 저장/실행 하여 디렉토리에 유저 파일을 생성. 근데 관리자 권한이 필요하니 sudo로 실행

- `id ops_user`를 통해 ops_user 생성 확인.
  
<img width="585" height="61" alt="Image" src="https://github.com/user-attachments/assets/9c76cda3-6e21-4817-84c9-3767a6749e13" />
