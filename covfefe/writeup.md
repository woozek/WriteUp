covfefe WriteUP
==========
index
------
+ 문제 설명
+ 사용한 도구 및 환경 세팅
+ 문제 해설

문제 설명
----------
이 문제는 가상머신(ova) 파일이 하나 주워지고 다른 vm으로 접근하여 flag 3개를 찾는 문제이다. 나는 칼리 리눅스와 우분투를 사용하여 문제를 해결했다.

사용한 도구 및 환경 세팅
-----------------------
+ 사용한 도구
    + kail(x64)
    + ubuntu(x64)
    + nmap
    + nickto
    + gdb (칼리, 우분투 내장)
    + John the Ripper (칼리 내장)


+  nat으로 ip 대역을 같게하여 접근해 문제를 해결했다.

문제 해설
---------

![nmap scan](./nmap.png)

두 가상 머신은 nat으로 설정했기 때문에 같은 대역이므로 nmap으로 스캔을 해봤다.
여기서 옵션들을 살펴보면 다음과 같다.

+ sS
    + TCP SYN 스캔
+ sV
    + 포트 어플리케이션 식별
+ T4
    + 속도 옵션 중에 가장 빠름

따라서 위 결과를 보면 192.168.248.136이 covfefe의 ip라는 것을 예측할 수 있다.
왜냐하면 31337 이라는 해커들이 많이 사용하는 수상한 port가 열려 있기 때문이다.

![nikto_scan](./nikto_command.png)

nikto 라는 취약점 분석 도구를 이용하여 192.168.248.136에 31337이라는 포트로 http가 열려있는 것을 확인할 수 있기 때문에 취약점 검사를 진행하여 result.html 이라는 파일로 만들었다.

![result_html](./result_html.png)

이처럼 취약점이 html 파일로 저장되어 시각화된 정보들을 볼 수 있다.
그래서 하나 하나 의심스러운 경로들을 들어가봤다.
get 방식으로 되어있기 때문에 url에 경로가 다 보인다.

![flag 1번](./flag1.png)

taxes로 이동해보니 첫번째 flag가 나왔다.

flag1{make_america_great_again}

![ssh_list](./ssh_list.png)

그리고 아까 ssh 접속을 시도해봤는데 접속이 되지않아서 넘겼었는데 .ssh/라는 경로가 있어 들어가보니 파일 3개를 다운 받을 수 있었다.

+ id_rsa
    + private key로 타인에게 노출되면 안되는 중요한 파일
+ id_rsa.pub
    + public key로 접속하려는 서버의 authorized_keys에 값이 저장됨
+ authorized_keys
    + 접속하려는 서버의 .ssh 디렉토리 아래의 위치, id_rsa.pub 키의 값을 저장하고 있음

>여기서 ssh의 원리를 생각해보면 알 수 있는데 사용자는 private key로 암호화하여 전송을하고 서버에서 그것을 다시 public key로 해독을 했을떄 값이 다르면 연결이 되지 않는다.

그래서 다운을받아 나는 kali에 ~/.ssh로 파일들을 옮겼다.

![ssh_id](./ssh_id.png)

그리고 is_rsa.pub 파일을 열어봤더니 simon@covfefe라는 문장을 보고 covfefe에 ssh계정 id는 simon 이라는 것을 알게 되었다.

그 후 다음과 같이 ssh로 접속을 해봤다.
> ssh simon@192.168.248.136

![access_error](./access_error.png)

이렇게 접속을 해보니 다음과 같이 암호를 입력하라고 나와서 막막했다.

![crack](./crack.png)

그래서 john the ripper라는 툴을 이용하여 ssh 키를 크랙했다.

> ssh2john 파일명 > 나올파일

이 명령은 john 형식으로 변환하여 나올파일로 저장하는 명령이다.

> john -show 나온파일

이 명령으로 john 형식에 파일에서 패스워드를 추출해낸다.

simon 비번 : starwars

![ssh_connact](./ssh_connact.png)

위와 같이 성공적으로 접속을 하였다.

![find](./find.png)

find 명령어로 setuid 즉, 실행중일 때 그 사용자 권한으로 사용되는 권한이 걸린 파일을 검색해봤다.

위에서 의심스러운 파일인 read_massage라는 파일을 볼 수 있다.

![read_massage](./read_massage.png)

그래서 다음 경로로가서 실행해봤다.

아무거나 입력했더니 오류가나서 Simon을 입력해보니 다음과 같은 문장이 나왔다.

home 디렉터리를 보라는데 /root 경로라고 생각할 수 있다.

![root_dir](./root_dir.png)

flag.txt와 read_massage.c라는 파일을 볼 수 있다.

그래서 flag.txt를 보려고 했더니 볼 수 있는 권한이 없었다.

그래서 그 옆에 read_massage.c 파일을 켜봤는데 2번째 플래그를 확인할 수 있었다.

flag2{use_the_source_luke}

코드를 쭉 읽어보면 아까 실행한 read_massage 프로그램에 c파일이라는 것을 알 수 있다.
그런데 gets 함수를 사용하고 있다. 따라서 bof 취약점이 발생한다.

맨처음 입력값을 Simon을 입력하지 않으면 프로그램이 종료되고, Simon을 입력하면 excuve 함수로 프로그램을 실행하는 것을 볼 수 있다.

따라서 bof 취약점을 이용하여 /bin/sh를 실행시키면 root 권한으로 실행되기 때문에 쉘이 따질 것이다.

따라서 배열크기 20을 넘기고 /bin/sh를 넣어주면 excuve에 /bin/sh가 들어갈 것이라는 시나리오를 세우고 해봤다.

![su](./su.png)

>Simon + 더미15개 + /bin/sh

성공적으로 쉘이 따졌다.

![flag3](./flag3.png)

flag.txt를 열면 flag다

flag3{das_bof_meister}
