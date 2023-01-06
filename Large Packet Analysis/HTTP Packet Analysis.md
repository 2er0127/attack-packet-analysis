HTTP를 이용한 응용 계층 공격은 트래픽이 워낙 적어 트래픽의 비율만으로는 공격 발생 여부를 알기 어렵다.

### 💡 httpry를 이용한 분석
httpry downloads ```$ brew install httpry``` <br/>

httpry는 http 요청/응답을 확인할 수 있는 도구이기에 패킷에 포함된 http 정보만 확인하고자 할 때 쉽게 사용 가능하다.<br/>
요청 값을 알아보기 위해 httpry에서 요청에 대한 방향성을 지시하는 문자열인 ```>```을 포함하는 값을 확인하는 명령어이다.

```$ httpry -r <file name> | grep ">"```

httpry의 실행 결과에서는 도메인 정보도 같이 확인이 되는데, 정적 컨텐츠를 포함한 다양한 파일들이 요청되어 모두 정상 요청처럼 보이므로 공격 발생 여부가 한 눈에 파악되지 않았다.

GET Flooding, POST Flooding과 같은 HTTP를 이용한 공격은 대부분 캐시 서버에서의 정적 컨텐츠 캐싱을 회피하려고 동적 컨텐츠만 대상으로 공격하는 특징이 있기 때문에 정적 컨텐츠를 제외하는 명령어를 작성해 본다.

```$ httpry -r <file name> | grep ">" | egrep -v "jpg|png|css|js|gif" | more```

옵션|설명
:---:|:---:
```egrep```|다수의 일치하는 문자열 지정.<br/> 문자열은 "" 또는 ''로 둘러싸야 하고, 각각의 문자열을 파이프(l)로 구분한다.

![httpry](https://user-images.githubusercontent.com/66156026/211017138-4687504f-4a0a-41f7-ad7d-6d2cd5e76039.png)

같은 도메인에 "/" 를 대상으로 단시간에 GET 요청이 발생한 것을 확인할 수 있고, 목적지 IP는 앞에서 확인한 IP와 같다는 것을 알 수 있다.

출발지 IP도 일정함을 보이고 있기 때문에 공격자 IP로 의심되는 192.168.0.20만 필터링하여 어떤 형태의 요청을 하고 있는지 점검해볼 수 있고, 시간 값을 제외한 모든 값을 정렬 후 카운팅 해볼 수 있다.

```$ httpry -r <file name> | grep ">" | grep <공격자 IP>```

```$ httpry -r <file name> | grep ">" | grep <목적지 IP> | awk '{print $3, $4, $5, $6, $7, $8, $9, $10}' | sort -n | uniq -c | sort -rn | more```

옵션|설명
:---:|:---:
```uniq -c```|중복 제거 후 한 줄만 출력하되, 같은 값이 얼마나 있었는지 카운트를 한다.
```sorn -rn```|내림차순 정렬이다. uniq -c 이후, 가장 높은 수치의 값을 위쪽으로 출력하기 위함이다.

<br/>

### 💡 ngrep를 이용한 분석
ngrep은 http를 비롯 UDP, ICMP 등 모든 유형의 프로토콜을 확인하기 쉬운 도구이다. 단순 요청/응답과 달리 헤더와 데이터까지 확인이 가능하므로 좀 더 세부적인 분석이 가능하다.

```$ ngrep -l <file name> -tWbyline http dst host <목적지 IP> | more```

옵션|설명
:---:|:---:
```-l```|저장된 pcap 파일 읽기
```-t```|시간값 출력
```-W```|출력 형식 지정<br/> - single: 각 패킷의 모든 항목을 구분없이 1줄로 출력 <br/> - byline: 각 패킷의 모든 항목을 구분하여 다른 라인으로 출력 <br/> - none: 각 패킷의 내용을 헤더 부분만 1줄로 출력
```http```|http만 출력하는 필터링 옵션<br/><br/> 다른 프로토콜은 proto라는 옵션을 함께 사용하여 필터링할 수 있다. <br/> - TCP만 출력: proto UDP <br/> - UDP만 출력: proto tcp <br/> - ICMP만 출력: proto ICMP
```dst host```|목적지 IP를 지정하여 필터링

<br/>

### 💡 http 트래픽만 분리하여 와이어샤크로 확인
이 공격 파일은 tcp의 비율이 적다는 것을 확인했기 때문에 tcp만 분리한다면 와이어샤크에서 패킷을 쉽게 볼 수 있을 것으로 생각된다.
대용량 파일을 tcpdump로 읽어 들여 tcp만 필터링 한 후 다른 .pcap 파일로 저장하는 명령어이다.

```$ tcpdump -nr <file name> tcp -w <new_file name>```

옵션|설명
:---:|:---:
```tcp```|tcp만 필터링
```-w```|캡쳐한 패킷을 다른 이름으로 저장

<img width="320" alt="capture" src="https://user-images.githubusercontent.com/66156026/211059894-1c378d01-c128-4a4b-b78d-8b0a95c33419.png">

tcp만 캡쳐한 파일은 530MB로 줄어든 것을 확인할 수 있다. 크기가 작기 때문에 와이어샤크를 이용해 분석이 가능하다.

---
만약 특이 사항으로 HTTP 헤더가 두 개로 보이고, HTTP 패킷이 Continuation이라는 형태를 보인다면 HTTP의 헤더 없이 데이터만 채워서 공격하는 경우를 말한다.

![wireshark_cap](https://user-images.githubusercontent.com/66156026/211061786-a482699a-0487-459c-ac9d-a791d9c52d43.png)

? 계속 요청을 보내고 404 응답을 받는 모습
