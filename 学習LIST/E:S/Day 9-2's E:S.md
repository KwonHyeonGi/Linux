# 📝 ERROR : Soultion

## E : ALB 서버 연결 안됌.
<img width="1670" height="771" alt="Image" src="https://github.com/user-attachments/assets/c88d163a-8ed5-4a3c-9dd4-7e206ca20f89" />



## S1 : TG[Target Group] 확인
<img width="914" height="768" alt="Image" src="https://github.com/user-attachments/assets/94252d2a-04d4-466d-8f7f-b62e95098dee" />

**이상 없음**

## S2 : ALB-SG 보안 그룹 확인
생성한 ALB-SG에 인바운드 규칙 ( HTTP 80 tcp 0.0.0.0/0) 이 없었음. 추가.
<img width="1635" height="768" alt="Image" src="https://github.com/user-attachments/assets/d755123a-064f-4f72-9ff0-478a4e08a19a" />

## 해결 완료.
