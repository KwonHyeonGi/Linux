# 📝LINUX 시스템 진단

## 흐름
> 부하 유발 및 진단
> 문제 프로세스 격리/종료
> 네트워크 상태 진단
> 메모리 부족 진단

### 부하 유발 및 진단
#### 서버에 의도적 부하를 걸기 위한 도구 `stress`
- `sudo apt update`
- `sudo apt install stress -y` : 여기서 -y는 자동 yes 응답
- `top` 을 통해 진단 도구 확인
- <img width="953" height="691" alt="Image" src="https://github.com/user-attachments/assets/b00768ff-17dd-405c-bd03-535d9cf9a4c7" />


#### CPU 부하 유발 
- `stress --cpu 4 --timeout 60s &` : stress(부하)를 cpu4개에 걸고 60초 후 자동 종료.
- <img width="1015" height="588" alt="Image" src="https://github.com/user-attachments/assets/20a3c819-5fbb-4a8b-ab1b-446d918afa43" />
- PID : 1651, 1652, 1653, 1654의 CPU부화량이 비정상적으로 높음을 확인. 원인은 명령어 `stress`

#### 문제 프로세스 격리 및 종료
- 식별된 부하의 원인(1651, 1652, 1653, 1654)를 격리하여 서버 정상화
- `kill` 명령을 통한 프로세스 강제 종료.
- ```
  kill 1651
  kill 1652
  kill 1653
  kill 1654
  ```
- <img width="903" height="207" alt="Image" src="https://github.com/user-attachments/assets/974f9316-183c-49a4-966a-224a85e2d10c" />
- stress 라는 프로세스를 통해서 4개의 cpu에 과부하를 걸게 했기에, 부조-자식 프로세스 모델로 작동됨.
- `stress: FAIL : [1650] (425) <-- worker 1651 got signal 15 ` : pid 1650 은 stress의 부모 프로세스 PID, kill 1651을 통해서 자식 프로세스에 SIGTERM(종료 요청)를 보냈고, 
그 신호를 받은 자식 프로세스가 종료되자 부모 프로세스(1650)이 전체 작업을 실패로 기록하며 종료했다는 메세지.
- `stress: WARN: [1650] (427) now reaping child worker processes` : 부모 프로세스(1650)가 나머지 다른 자식 프로세스를 정리하고 있다는 경고.
- 그렇기에 나머지 PID는 kill을 해도 no such process라고 메세지가 나옴.
- <img width="1000" height="598" alt="Image" src="https://github.com/user-attachments/assets/3ab4cfdf-b799-4a9c-9909-1b4eb1f39d39" />


* * *

### 네트워크 상태 진단
#### 웹 서버 설치 및 상태 확인
- 웹 서버 설치 `sudo apt install nginx -y`
- `ss tuln` : [t]: TCP 소켓, [u] : UDP 소켓, [l] : LISTEN 상태의 소켓, [n] : 서비스 이름 대신 포트 번호
- 0.0.0.0:80 이 LISTEN 상태임을 확인 | LISTEN : 대기 상태 | ESTAB : 연결 상태 | CLOSE-WAIT : 종료 대기 상태
- <img width="1075" height="276" alt="Image" src="https://github.com/user-attachments/assets/2d9bc8db-d1bb-469e-b61d-0f24b470c44f" />

#### 포트 충돌  
- `sudo python3 -m http.server 80` : 80번 포트를 사용하려는 시도
- <img width="662" height="366" alt="Image" src="https://github.com/user-attachments/assets/305893f7-53c3-4234-bbc3-1a7ff7122657" />
- 포트 충돌 확인

#### 충돌 원인 프로세스 식별
- `sudo ss -ptln` : [p] : 프로세스 정보 표시, [t] : TCP 소켓 표시, [n] : 포트/ip를 숫자로 표시, [l] : LISTEN 상태만 표시
- <img width="1101" height="283" alt="Image" src="https://github.com/user-attachments/assets/caf99fba-73a0-4af1-acec-82335252c8d5" />
- nginx로 인한 것을 확인

#### 트러블 슈팅(해결)
- `sudo kill 1899` 
- `sudo python3 -m http.server 80` : 80번 포트를 사용하고있던, nginx를 종료, 해제 했으니 python 웹 서버로 80번 포트 사용 해보기
- <img width="1065" height="679" alt="Image" src="https://github.com/user-attachments/assets/077b8465-2454-4842-875b-2869d7e6ff3e" />
- Nginx가 포트를 놓지않고 재시작함.
- 🔴 systemd의 자동 복구 기능 : /ect/systemd/system/에 서비스 파일이 등록되, 항상 실행상태를유지하도록 설정되어있음. kill 1899로  nginx는 종료되었지만, systemd가 비정상 종료로 간주하고 새로운 pid로 nginx를 재시작함.
- ✔️`sudo systemctl stop nginx` :  직접적으로 systemctl에게 nginx를 정지시키라고 지시.
- `sudo python3 -m http.server 80`
- <img width="592" height="53" alt="Image" src="https://github.com/user-attachments/assets/49b11e03-eb2c-43a1-8fc1-97d930e1cc5e" />

* * * 

### 메모리 및 디스크 진단
#### 메모리 사용 현황 확인(free)
- `free -h`
- <img width="731" height="73" alt="Image" src="https://github.com/user-attachments/assets/30702993-588b-48c3-8418-56d75994f50a" />
- `-h` : gb, mb로 표시형식 변경
- Mem(RAM) | total(전체) | used(사용) | free(남은 양)
- Swap : 하드디스크의 일부를 메모리로 사용하는 영역.
- Mem used가 90%에 가까워질 수록 문제가 있음.

#### 디스크 사용 공간 확인(df)
- `df -h`
- <img width="598" height="185" alt="Image" src="https://github.com/user-attachments/assets/5624196a-badd-4c9b-954e-206d4546e835" />
- Filesystem : 파일 시스템 이름 | Size : 전체 크기 | Avail : 남은 공간
- /dev/root (루트 파일 시스템) 의 Use(사용률)이 80%이상일 시 디스크 부족 문제가 발생할 수 있음.
