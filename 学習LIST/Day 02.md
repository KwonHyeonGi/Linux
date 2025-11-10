# 📝 Keyword [ Redirection, Input, Output, Shell, Shell script ]

## Redirection : 방향 전환   

## output : 출력
- `Standard Output : 1>` : 기본 출력
- `Standard Error : 2>` : 에러값 출력

<img width="722" height="209" alt="Image" src="https://github.com/user-attachments/assets/14226b77-e7f6-46a0-9fda-eb3bb46dfa7b" />
  
## input : 입력
- `Standard Input : < ` : 입력을 해주는 명령어	

## append
- ` >> ` : 출력 값을 더한다

<img width="549" height="497" alt="Image" src="https://github.com/user-attachments/assets/8b3ed66c-4c52-4bb3-bc11-573229129725" />

## Shell  
- `하드웨어 > 커널 > 쉘` : 쉘에게 명령을 내리면 커널이 이해할 수 있게 쉘이 커널에게 전달해줌 그리고 명령이 하달됌

## Shell script
- `명령어를 저장 하는 시스템`
```
  bak이라는 디렉토리를 만들고, 디렉토리에 1.log 2.log 3.log를 만든다 라는 명령어를 저장 하고 싶을 경우
  nano를 통해서
    #! /bin/bash
    if ! [ -d bak  ] ; then
    	mkdir bak
    fi
    cp *.log bak
  이러한 if문을 적어준다.
    : bak라는 디렉토리가 없을경우 mkdir bak 하고 *.log를 카피해서 bak에 저장한다(여기서 *은 1,2,3.log를 다 지정해준 것)

  그 후 터미널을 통해서 이 파일을 실행할 경우, if문 대로 명령되는 것을 볼 수 있음.
```
