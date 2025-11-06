# Day 2. Keyword [ Redirection, Input, Output, Shell, Shell script ]

**Redirection**: 방향 전환  
 input ->  명령 -> output(결과)
 
* * *

**output** : 출력   
'>' : 출력을 해주는 명령어  
**ex)** ls -l > result.log  :  ls -l 을 result.log에 출력 저장. 
   
    re rename2.txt 후 다시한번 rm rename2.txt 하면 에러가 나옴
    그 에러 내용을 다이렉션하여 error.log라는 파일에 출력
    : rm rename2.txt 1> result.txt 2> error.log

   출력에는 1> 2>가 있음.
     1>는 성공시 아무런 메세지가 안뜸 , 하지만 실패일 경우 에러 메세지가 나옴
     2> 는 표준에러를 출력해주는 명령어

    그렇기에 rm rename2.txt를 했을때 rename2라는 파일이 없을경우 파일삭제가 실패, 즉 에러가 발생하기에 2>(표준에러)에 출력되게됨.

* * *

**input** : 입력
   < : 입력을 해주는 명령어
    head n1 < linux.txt > one.txt
    linux.txt라는 파일의 전체에서 첫번째 행만 one.txt에 출력해라
	
* * *

**append** 
  >> : 더한다
    출력값을 더한다는 거임
    ls -al >> result.txt : result.txt와 result.txt를 더해서 두번 보여줌
    
    /dev/null  : 휴지통 같은 역할

* * *

**Shell** 
  shell과 kernel
  하드웨어 > 커널 > 쉘
  쉘에게 명령을 내리면 커널이 이해할 수 있게 쉘이 커널에게 전달해줌 그리고 명령이 하달됌

 * * *
 
**Shell script**
  명령어를 저장 하는 시스템

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
