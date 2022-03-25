# ⚖️ UDP port scan
UDP는 3-way-handshake가 없는 **신뢰성 없는 통신**이지만 수신 측의 포트가 닫혀있을 때는 **ICMP가 응답**하기 때문에 상태 값 정도는 확인할 수 있다.

열려 있는 포트로부터는 특정 UDP 응답 값이 수신되고, **닫혀 있는 포트**로부터는 ```ICMP port Unreachable``` 패킷이 수신되며, 방화벽과 같은 차단 정책에 차단되었을 때는 어떠한 패킷도 응답하지 않는다.

포트 상태|공격자|공격 대상
:---:|:---:|:---:
열린 포트|```UDP```|```UDP```
닫힌 포트|```UDP```|```ICMP port Unreachable```
차단 정책|```UDP```|```-```

Nmap을 이용한 UDP 포트 스캔의 기본 명령어는 ```nmap -sU [대상 서버 IP]``` 형식이다.

TCP 포트 스캔과는 다르게 다수의 ICMP port Unreachable 패킷이 확인된다.

![스크린샷 2022-03-25 오후 7 12 21](https://user-images.githubusercontent.com/66156026/160101317-65b3c3bb-c121-4118-993f-831fb384b6d1.png)

