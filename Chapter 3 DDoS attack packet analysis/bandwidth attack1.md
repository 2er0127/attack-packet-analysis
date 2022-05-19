# ⚔️ 대역폭 공격
대역폭 공격은 공격 대상의 네트워크 회선 대역폭을 고갈시켜 더 이상의 정상 사용자가 접속하지 못하게 만드는 공격이다.
공격자는 대규모 트래픽을 성생하기 위해 위조가 쉬운 UDP나 ICMP를 주로 사용하고, 패킷의 크기를 가능한 한 크게 위조하여 전송한다.

## 💡 UDP Flooding
UDP는 **비연결형** 프로토콜이자, 데이터 전송 시 오류가 발생하더라도 재전송을 하지 않는 신뢰성 없는 프로토콜이다. 
빠른 속도가 우선이 되는 실시간 방송이나 데이터가 유실되어도 큰 지장이 없는 데이터를 전송할 때 사용한다.

UDP는 패킷 구조가 단순하고 3-way-handshake와 같은 검증 절차가 없어서 패킷의 크기와 출발지 IP도 위조가 가능하기 때문에 **UDP를 이용한 공격에서는 출발지 IP로 공격자를 추적하는 것은 옳지 않다.**

<img width="430" alt="udp-header" src="https://user-images.githubusercontent.com/66156026/159487814-e44a008c-1ff9-45c8-8288-3b981f2009b2.png">

### 🧸 UDP Flooding의 패킷 분석
![스크린샷 2022-04-04 오후 11 34 12](https://user-images.githubusercontent.com/66156026/161567220-92a6369c-2c9e-4ede-afe0-eef67c82fd7b.png)

프로토콜 ```UDP```, 패킷의 크기는 ```1442```이다. 패킷의 크기가 상당히 큰 것을 알 수 있고, 데이터를 보면 '```xxxx```' 형태의 의미 없는 값임을 알 수 있다.
**패킷을 의도적으로 크게 만들기 위한 것**이라고 간주할 수 있다.

또한 목적지가 같은 여러 패킷에 모든 출발지 IP가 무작위로 다른 것을 보면 **위조한(Spoofing) 상태**라고 생각할 수 있다.
대부분 자신의 IP를 숨기기 위함이고, Anti-DDoS와 같은 차단 장비에서 임계치 차단 정책에 차단되지 않게 하기 위해 출발지 IP를 위조하는 것이 당연시된다.
그렇기 때문에 UDP와 ICMP를 이용한 공격은 *대부분 출발지 IP가 위조되었을 것*이라 생각하는 게 좋다.

애초에 UDP 프로토콜을 사용하면서 공격 대상 포트가 ```80번 HTTP```로 지정되어 있다. **UDP를 포트 번호 80으로 요청했다는 것 자체로 비정상 요청**이라 볼 수 있으며,
대역폭 공격은 회선의 대역폭을 가득 채우는 것이 목적이기 때문에 **목적지 포트가 어디로 지정되었는지는 공격의 영향도와 관련이 없다.**

> ```QUIC```라는 프로토콜은 UDP를 이용하여 포트 번호 **80** 또는 **443**으로 통신한다. <br/>
> 그러나 **이런 특수한 프로토콜이 자신이 운영 중인 네트워크에서 서비스되고 있지 않다면 UDP로 80/443 통신은 없는 것과 마찬가지로 생각하는 것이 맞다.**

### 🧸 UDP Flooding 대응 방안
(대부분의 대역폭 공격에도 해당하는 대응 방안일 수 있다.)

**1. 충분한 네트워크 대역폭 확보**

모든 대역폭 공격은 회선 대역폭을 고갈시키는 공격 유형이므로 정상적인 대응을 위해서는 공격 트래픽보다 더 큰 규모의 회선 대역폭을 보유하고 있어야 한다.
충분한 회선 대역폭을 보유하고 있다면 상단 라우터 또는 차단 장비에서 공격 트래픽을 필터링하고 정상적인 트래픽만 서버로 전송하여 서비스를 유지할 수 있다.

하지만 무작정 회선 대역폭을 늘리는 것은 불가능하기 때문에 직접적인 방안이 될 수는 없다. 큰 네트워크 회선 대역폭을 보유한 ISP나 CDN 업체들이 이러한 방법으로 DDoS 방어 사업을 하고 있다.

**2. 미사용 프로토콜 필터링**

운영 중인 네트워크와 서비스의 구조가 UDP를 사용하지 않는 환경이라면 상단 라우터 또는 차단 장비에서 UDP를 차단하는 것만으로 UDP Flooding의 원천적인 대응을 할 수 있다.
하지만 일반적으로 DNS, SNMP, NTP 등의 UDP 프로토콜을 대부분 기본적으로 사용하고 있기 때문에 해당 53, 161, 123 등의 사용 중인 포트만 허용 후 나머지 포트들을 대상으로 UDP를 차단하거나, 신뢰할 수 있는 특정 호스트를 제외한 나머지 IP에 대해 UDP를 차단 설정할 수 있다.

하지만 이 설정도 상단 라우터의 대역폭 이상으로 공격이 발생한다면 무용지물이 된다. 만약 운영 중인 네트워크에 DNS와 같은 UDP를 이용한 서비스가 운영되고 있다면 해당 서비스에는 영향이 발생하지 않도록 해야하므로 차단 설정이 까다로워진다.
가장 우선으로 할 수 있는 방안은 UDP 서비스를 사용하지 않는 목적지 IP와 대역으로 UDP를 차단하는 것이다.

이후에는 UDP 서비스 서버의 IP에 대해 임계치 기반의 차단 설정이나 단편화 패킷 차단, 특정 크기 이상의 패킷 차단 등의 정책을 적용하여 좀 더 세부적인 차단 정책을 적용해야 한다.

**3. 위조된 IP 차단**

IP 위조가 쉬운 공격 유형들은(UDP, ICMP, SYN Flooding 등) 대부분 무작위로 IP가 위조되기 때문에 공인 IP로 통신하는 네트워크에서는 사용 불가능한 IP들이 공격에 사용되기도 한다.

공인 IP로 사용될 수 없는 IP(Bogon IP or Martian packet), 존재할 수 없는 형식의 비정상 IP들이 해당되며 상단 라우터의 인입 구간에 ACL을 적용하여 차단하거나 Anti-DDoS와 같은 차단 장비에서 제공하는 비정상 IP 차단 기능 등을 이용하여 차단할 수 있다.

**4. 출발지 IP별 임계치 기반 차단**

공격자가 단일 IP로 공격을 시도했다면 출발지 IP별 임계치 차단 기능을 이용하여 차단할 수 있다. 또는 위조된 IP라고 하더라도 특정 IP 대역 내에서 공격을 발생시키다 보면 특정 출발지 IP가 반복적으로 사용되기도 하므로,
이 기능에 의해 차단될 가능성도 일부 존재한다.

**5. 패킷 크기 기반 차단**

출발지 IP가 잘 위조되어 임계치로도 차단할 수 없고, 비정상 IP 차단 기능으로도 대응할 수 없을 때는 특정 크기 이상의 UDP 패킷을 차단하는 방안도 고려해 볼 수 있다.
대부분의 대역폭 공격 유형들이 대체로 패킷의 크기를 비정상적으로 크게 위조한 점을 고려한 대응 방안으로, 차단 설정을 위한 패킷 크기의 산정은 운영 중인 UDP 서비스의 패킷 크기를 모니터링하여 오탐 가능성을 고려하여 충분한 이상 값으로 설정하는 것이 좋다.

**6. Fragmentation 패킷 차단**

패킷의 크기를 늘리다 보면 MTU 이상의 크기로 전송되어 패킷이 단편화 되는 경우가 발생한다.
UDP 서비스가 운영되는 네트워크 환경에서도 단편화가 될 만큼 큰 크기의 UDP 패킷이 통신되는 경우가 거의 없기 때문에, UDP Fragmentation 패킷을 차단하는 설정도 좋은 방안이 된다.
Fragmentation 패킷 차단을 적용하기 이전에 단편화된 패킷이 통신되지 않는지 확인 후 적용해야 한다.

**7. 서버 대역폭 및 가용량 확대**

UDP는 실제 UDP 서비스에 사용되는 패킷과 거의 유사하게 위조할 수 있다. 약 60~70byte의 작은 크기로 무작위의 IP를 이용하여 공격이 발생한다면, 패킷의 크기나 임계치로는 차단할 수 없는 상황이 발생한다.

대역폭 공격의 의도와는 달리 큰 트래픽을 생성할 수는 없지만 많은 패킷이 전송되어 서버의 부하를 발생시킬 수 있기 때문에 서버는 최소한의 부하를 견뎌낼 수 있는 성능을 보유하고 있어야 하며, 많은 대역폭까지는 아니더라도 네트워크 인터페이스를 본딩(bonding)하여 2~4Gbps까지는 증대시켜 일부 트래픽이 유입되더라도 견딜 수 있는 가용성을 확보해야 한다.

**8. Anycast를 이용한 대응**

Anycast는 BGP 라우팅 프로토콜을 이용하여 여러 네트워크에서 같은 IP 대역을 광고하여 같은 IP를 여러 네트워크에서 사용할 수 있게 하는 네트워크 기법이다.
사용자는 자신이 위치한 네트워크에서 가장 가까운 거리의 노드로 접속하게 되므로 속도의 개선뿐만 아니라 트래픽을 분산하는 효과도 얻을 수 있다.

Anycast로 광고 중인 여러 노드 중 한 개의 노드에 장애가 발생할 경우, 사용자는 또 다른 근거리의 정상 노드로 접속하여 통신을 유지할 수 있다. 특정 노드에 접속 불가 현상이 발생할 때는 평소 사용하던 노드보다 먼 거리의 노드로 접속되어 평소보다 지연 현상을 느낄 수도 있지만 정상적인 운영으로 간주할 수 있다.

> ```BGP(Border Gateway Protocol)``` <br/>
> ```BGP```란 AS라는 단일 네트워크 또는 네트워크 그룹을 단위로 하여 이웃하는 AS들끼리 TCP를 이용하여 라우팅 정보를 공유하는 라우팅 프로토콜이다. <br/><br/>
> 라우팅 경로 설정 시 AS 간의 경로가 가장 짧은 경로를 최선의 경로로 라우팅 테이블을 설정하며, 하나의 인터넷 경로가 다운되었을 때 다른 연결이 가능한 경로로 패킷을 재전송하여 네트워크의 안정성을 제공하므로, 모든 서비스 IP가 같은 Anycast 방식의 통신에서 특정 서버가 다운되면 또다른 가장 짧은 거리에 구축된 서버로 연결을 가능하게 해준다.

**9. Ingress 필터링과 Egress 필터링**

이 기능은 DDoS 공격의 차단 기능보다는 예방 기능이라고 볼 수 있다. 연결된 라우터 상호 간에는 사용하는 IP 대역을 알고 있기 때문에 전송되는 킷의 출발지 IP가 상대방 라우터에서 사용 가능한 IP 대역이 아닌 경우 차단하는 기능을 의미한다.

이 기능을 사용하면 자신의 네트워크에서 사용할 수 있는 출발지 IP의 패킷만 전송되므로, 자신의 네트워크 내에서 외부로 발생하는 IP 위조 공격을 원천적으로 예방할 수 있고 만약 이 설정이 전 세계 네트워크에 설정된다면 위조된 IP를 이용한 DDoS 공격은 원천적으로 불가능해질 것이다.

외부의 네트워크에서 내부로 들어오는 패킷에 차단하는 설정을 ```Ingress``` 필터링이라 하고, 내부의 네트워크에서 외부로 나가는 패킷에 차단하는 설정을 ```Egress``` 필터링이라고 한다.

![ine](https://user-images.githubusercontent.com/66156026/161801963-39b723d9-a4cd-42d9-9d1b-ccb647bbc201.png)

수동으로 ACL 설정을 하여 Ingress 필터링과 Egress 필터링을 설정할 수도 있으며 운영자의 계획에 따라 인입 구간 또는 외부 전송 구간에 선택적으로 적용할 수 있다.
사용 가능한 IP 대역의 관리와 ACL 적용만 잘 이루어진다면 자신이 관리 중인 네트워크에서 출발지 IP가 위조되어 비정상 트래픽이 발생하는 것을 방지할 수 있다.

이런 Ingress 필터링과 Egress 필터링을 적용하여 위조된 IP를 사용 불가능하게 하는 조치는 Real IP를 이용하는 응용 계층 공격을 제외한 나머지 형태의 공격(대역폭/자원 고갈 공격 등)을 예방하는 데 엄청난 효과가 있다고 할 수 있다.

> **네트워크 점검 도구** ```hping``` <br/>
> 주로 네트워크 점검을 목적으로 사용하는 패킷 생성 도구이며 TCP, ICMP, UDP 등 다양한 패킷 생성이 가능하기 때문에 DDoS 방어 용도로 수립한 차단 정책을 점검하기 위한 목적으로도 사용할 수 있다. <br/>
> 옵션을 사용하여 ICMP, UDP, TCP Flags를 이용한 Flooding을 생성할 수 있으며, 인가된 네트워크를 대상으로 점검 용도로만 사용해야 한다.

옵션|설명
:---:|:---:|
```-1```|공격 모드를 선택하는 옵션. <br/> ```-1``` : ICMP mode <br/> ```-2``` : UDP mode <br/> ```-3``` : SCAN mode <br/> ```-S``` : TCP SYN 전송(R/P/A/U : RST/PSH/ACK/URG)
x.x.x.x|목적지 IP
```-s```|출발지 포트 지정(지정하지 않으면 무작위 값으로 생성)
```-p```|목적지 포트 지정(지정하지 않으면 무작위 값으로 생성)
```-d```|데이터 크기 지정
```-i```|패킷 전송 주기이며, 마이크로초 단위이다. <br/> ```-u10000```(1초에 10 패킷 생성) <br/> ```-u100```(1초에 1000 패킷 생성)
```-c```|전송할 패킷 수 지정(지정하지 않을 경우, 지속 발생)
```--rand-source```|출발지 IP를 무작위로 변경
```--flood```|최대 성능으로 패킷 생성

ex) ```hping3 -1 x.x.x.x -s 1900 -p 80 -d 8000 -i u10 -c 100000 --rand-source -flood```

<br/>
<br/>

## 💡 ICMP Flooding
**ICMP(Internet Control Message Protocol)** 는 인터넷 환경에서 오류에 관한 처리를 지원하는 역할을 한다.
라우터와 같은 네트워크 장비에서 패킷을 전송할 때, 해당 패킷이 목적지 호스트까지 도달하지 못하거나 목적지 호스트가 정상적으로 동작하지 않을 때 에러 메시지를 응답하여 오류 상황을 알려준다. 
가장 흔히 아는 ```ping``` 명령어를 예로 들 수 있다.

![ICMP 패킷 구조](https://user-images.githubusercontent.com/66156026/167421731-a074ae20-ab3c-4bf2-9a1d-d4dcd261899d.png)

오류에 관한 내용은 Type과 Code의 값으로 구분한다.

#### 자주 사용하는 ICMP 메시지 타입

Type|Code|Message
:---:|:---:|:---:|
0|0|Echo reply
3|0|Destination network unreachable
3|1|Destination host unreachable
3|2|Destination protocol unreachable
3|3|Destination port unreachable
3|4|Fragmentation required, and DF flag set
3|6|Destination network unknown
3|7|Destination host unknown
4|0|Source quench(congestion control)
8|0|Echo request(used to ping)

### 🧸 ICMP Flooding 공격 패킷 분석

![스크린샷 2022-05-09 오후 10 42 51](https://user-images.githubusercontent.com/66156026/167423182-6d7b2c02-e72b-4c7b-a467-be0730a8060b.png)

ICMP 요청 패킷을 이용하여 공격을 발생시키기 때문에 ICMP 메시지 타입에서 ```Type 8, Code 0```에 해당하는 **Echo request**가 사용되는 것을 확인할 수 있다.
데이터 크기가(Length) **1442**로 상당히 큰 크기이며, 데이터 값이 '```XXXXX```' 형태의 의미없는 값이다. 패킷의 크기를 의도적으로 크게 만들기 위해서 라는 것을 알 수 있고,
출발지 IP 또한 무작위 형태인 것으로 보아 **위조한(Spoofing) 상태라고 간주**할 수 있다.

### 🧸 ICMP Flooding 대응 방안
ICMP Flooding은 UDP Flooding과 마찬가지로 회선 대역폭을 고갈시키는 대역폭 공격이다. 그렇기 때문에 UDP Flooding의 대응 방안과 거의 비슷하다.

- 충분한 네트워크 대역폭 확보
- 위조된 IP 차단
- 출발지 IP별 임계치 기반 차단
- Fragmentation 패킷 차단
- 서버 대역폭 및 가용량 확대
- Anycast를 이용한 대응
- Ingress 필터링과 Egress 필터링

**미사용 프로토콜 필터링** <br/>
UDP와 달리 ICMP는 헬스 체크의 목적으로만 사용할 뿐 서비스 목적으로는 사용하지 않는 프로토콜이다. 상단 라우터 또는 차단 장비에서 차단하더라도 서비스 운영에는 거의 영향이 없다.
하지만 ```ping, traceroute, mtr```과 같은 ICMP 명령어를 사용하기 때문에 단지 프로토콜을 차단하는 것은 부담이 될 수 있다. DDoS 방어를 목적으로 ICMP 프로토콜을 사용하지 않기를 감수한다면 차단 장비에서 원천 차단하여 대응할 수 있다.

**패킷 크기 기반 차단** <br/>
특정 크기 이상의 ICMP 패킷을 차단해 두면 큰 패킷으로 위조된 형태의 ICMP Flooding을 차단할 수 있다.
일반적인 ping이나 traceroute에서 사용되는 ```ICMP echo request```의 경우에는 **64~80바이트** 정도의 패킷으로 전송되지만,
응답되는 ```ICMP echo reply```의 경우 **약 300바이트**가량의 큰 크기로 수신될 수도 있어 자신의 네트워크에서 통신되는 ICMP 패킷의 크기를 조사하여 **임계치를 설정**하는 것이 좋다.

<br/>
<br/>

## 💡 IP Flooding
**Layer 3**에 속하는 IP 프로토콜을 이용한 공격 형태이며 UDP와 ICMP의 상위 프로토콜이다. IP 프로토콜에서는 프로토콜 번호로 각 프로토콜을 구분할 수 있다.

**잘 알려진 프로토콜**

Hex|Protocol Number|Keyword|Protocol
:---:|:---:|:---:|:---
```0x01```|```1```|```ICMP```|```Internet Control Message Protocol```
```0x06```|```6```|```TCP```|```Transmission Control Protocol```
```0x11```|```17```|```UDP```|```User Datagram Protocol```
```0x2F```|```47```|```GRE```|```Generic Routing Encapsulation```

공격의 관점에서 보면 IP 프로토콜을 사용한다는 것 이외에 일반적인 대역폭 공격과 큰 차이점이 없다. 공격의 특징과 대응 방안도 비슷하다.
<br/> IP 패킷을 공격 대상에게 일방적으로 전송하여 대역폭을 고갈시키는 공격 유형이며 패킷의 위조가 가능하기 때문에 출발지 IP를 비롯한 데이터 값을 위조하여 큰 크기의 패킷을 공격에 사용한다.

### 🧸 IP Flooding 공격 패킷 분석
![스크린샷 2022-05-15 오후 6 07 40](https://user-images.githubusercontent.com/66156026/168465420-aeec21aa-775b-4345-9935-b2bbcf4e534f.png)

출발지 IP를 위조할 수 있지만 위 공격 패킷에서는 중복된 출발지 IP가 보인다. **IP 헤더 내의 프로토콜 값을 무작위로 변경**하고 패킷의 크기 또한 크게 위조한 형태이다.

### 🧸 IP Flooding 대응 방안
- 충분한 네트워크 대역폭 확보
- 출발지 IP별 임계치 기반 차단
- Fragmentation 패킷 차단
- 서버 대역폭 및 가용량 확대
- Anycast를 이용한 대응
- Ingress 필터링과 Egress 필터링

**미사용 프로토콜 필터링** <br/>
IP 프로토콜은 TCP와 UDP의 상위 계층이므로 **무작정 차단하게 되면 하위 계층에 속하는 모든 프로토콜이 사용 불가능**해진다.
그렇기에 자신의 네트워크에서 사용하는 프로토콜을 허용해 준 후 IP 프로토콜을 차단하는 설정을 할 수 있다. <br/>
```ACL 설정```은 허용을 위한 IP 대역과 프로토콜, 세부적으로는 포트 등을 정의 후 마지막에는 **이외의 모든 IP 프로토콜을 차단하는 설정**이기 때문에 자신의 네트워크가 어떤 IP 대역을 사용하는지, 어떤 서비스가 운영되고 있는지 정확히 알고 있는 상황에서 시도해야 한다. <br/>
> **특정 프로토콜 허용 후 IP 프로토콜을 차단하는 예시**
> ``` 
> Router#show access-list
> Extended IP access list 101
>   10 permit tcp any 1.1.1.0 0.0.0.255
>   11 permit tcp any 2.2.2.0 0.0.0.255
>   20 permit icmp any 1.1.1.0 0.0.0.255
>   30 permit udp any 1.1.1.248 0.0.0.7 eq 53
>   40 permit udp 8.8.8.8 any
>   41 permit udp 168.126.63.1 any
>   ...
>   ...
>   99999 deny ip any any
> ```

<br/>
<br/>

## 💡 Fragmentation Flooding
Fragmentation Flooding은 네트워크 기기가 전송할 수 있는 최대 전송 단위인 ```MTU(Maximum Transfer Unit)``` 이상의 크기로 패킷을 전송하여 패킷이 분할되는 **단편화(Fragmentation)** 의 특징을 이용한 공격 유형이다.

> **MTU(Maximum Transfer Unit)** <br/>
> 네트워크 기기가 전송할 수 있는 최대 전송 단위. 네트워크 환경에 따라 각각 다르지만, 현재 대부분의 네트워크 환경에 사용 중인 이더넷의 MTU는 1500bytes이다. <br/>
> <br/>**단편화** <br/>
> MTU 이상의 크기로 패킷을 전송하게 되면, 패킷이 MTU 크기에 맞춰 분할된다.

**TCP에서 단편화를 회피하기 위한 기법** <br/>
단편화의 정보를 나타내는 Flags 헤더는 IP 프로토콜에 해당한다. 그렇기에 IP 프로토콜 위에 존재하는 모든 프로토콜(UDP, ICMP, TCP)은 단편화가 발생할 수 있다.
<br/>UDP와 ICMP는 비신뢰성 통신 방식을 사용하기 때문에 중간에 패킷 하나가 유실되어도 큰 문제가 되지 않는다.
그러나 TCP는 신뢰성 있는 통신 방식을 사용하기 때문에 패킷 하나가 유실되었을 때 어느 단편화된 패킷이 유실되었는지 확인하고 재전송하는 과정에서 많은 리소스와 시간이 소모되게된다.
<br/>결국 효율적인 통신을 위해 TCP는 단편화를 하지 않는 방법이 필요했고 그 해결책이 **세그멘테이션**이다.

> **세그멘테이션(Segmentation)** <br/>
> TCP는 패킷이라는 단위로 불리지만 엄밀히 말하자면 세그먼트라는 단위이다. OSI 7계층에서 Layer4에 해당한다.

TCP 세그먼트의 크기가 MTU를 초과한다면 IP 패킷으로 캡슐화되어 전송될 때 단편화가 발생하게 되는데, 이때 발생할 수 있는 단편화를 회피하고자 MTU를 초과하지 않는 크기로 세그먼트를 분할하는 방법을 **세그멘테이션**이라고 한다.
이를 위해 ```MSS```와 ```PMTUD```라는 기법이 사용된다.

수신 측에서 세그멘테이션된 TCP 세그먼트들을 재조합하는 방식은 sequence number, ack number, window size, checksum 등의 헤더 정보를 이용하므로 TCP 레벨에서 재조합이 가능하다.

**MSS(Maximum Segment Size)** <br/>
클라이언트와 서버 간의 TCP 통신 시 최대로 전송할 수 있는 TCP 세그먼트의 크기를 의미한다. <br/>
3-way-handshake 간에 사용되는 SYN, SYN-ACK 패킷의 option 헤더에 명시가 되는데, 그 크기는 실제 전송되는 데이터의 크기이므로 MTU에서 헤더의 크기인 40bytes(IP 헤더 20bytes + TCP 헤더 20bytes)를 뺀 나머지 값이다.
<br/>현재 대부분의 네트워크는 이더넷 환경으로 MTU가 1500bytes이므로, 대부분의 MSS의 값도 1460bytes이다.

<img width="885" alt="스크린샷 2022-05-19 오후 1 52 37" src="https://user-images.githubusercontent.com/66156026/169210654-2f89124e-4870-463e-adc4-0ef4d483bd04.png">

**TCP 통신 시, 호스트 간 MSS 크기 설정 과정** <br/>
<img width="577" alt="스크린샷 2022-05-19 오후 2 13 16" src="https://user-images.githubusercontent.com/66156026/169213553-f279107b-a895-4944-a67a-0c04456b49cf.png">
1. 클라이언트는 자신의 MSS Buffer(16K)와 MTU 사이즈 - 40bytes(1500-40=1460)를 비교 후, 더 낮은 값으로 MSS를 설정하여 SYN 패킷을 전송한다.
```SYN, MSS = 1460```
2. 서버는 클라이언트로부터 수신한 MSS(1460)를 자신의 아웃바운드 인터페이스의 MTU 사이즈 - 40bytes(4422)와 크기를 비교한다.
```SYN, MSS = 1460```
3. 서버는 상호 간의 MSS를 비교 후, 작은 크기의 값으로 MSS(1460)를 설정한다.
```Set "Send MSS" = 1460```
4. 서버는 자신의 MSS Buffer(8K)와 MTU 사이즈 - 40bytes(4462-40=4422)를 비교하여 더 낮은 값으로 MSS(4422)를 설정하여 SYN-ACK을 전송한다.
```SYN, ACK MSS = 4422```
5. 클라이언트는 서버로부터 수신한 MSS(4422)와 자신의 아웃바운드 인터페이스의 MTU 사이즈 - 40bytes(1460)의 크기를 비교한다.
```SYN, ACK MSS = 4422```
6. 클라이언트는 상호 간의 MSS를 비교 후, 작은 크기의 값으로 MSS(1460)를 설정한다.
```Set "Send MSS" = 1460```

<br/>

**PMTUD(Path MTU Discovery)** <br/>
TCP 통신 시 두 호스트 간에는 MSS를 이용하여 MTU를 초과하지 않는 값으로 데이터를 전송하였다.
그런데 만약 패킷이 전송되는 경로(Path) 상의 라우터에서 자신보다 작은 크기의 MTU를 사용하는 라우터가 존재한다면, 단편화는 라우터 구간에서 발생하게 되는데,
전송 경로 상에서의 단편화를 회피하려고 사용하는 기능이 PMTUD이다. <br/>
PMTU(Path MTU)는 라우팅 경로 상의 MTU를 의미하고, PMTUD는 네트워크 경로 상의 존재하는 수많은 네트워크 장비들이 상호 간의 MTU를 미리 파악(Discovery)하여 단편화를 회피할 수 있도록 하는 기능이다.
최근 대부분의 라우터 및 단말 기기에 기본적으로 PMTUD가 활성화되어 있다. <br/>
PMTUD가 활성화된 장비들은 기본적으로 TCP에 해당하는 패킷에게는 Flags 헤더를 '```Don't Fragment(DF)```'로 설정하여 전송하기 때문에 애초부터 단편화가 발생하지 않게끔 한다.

**PMTUD의 흐름** <br/>
<img width="577" alt="스크린샷 2022-05-19 오후 2 28 06" src="https://user-images.githubusercontent.com/66156026/169217043-4703a6d3-6767-43cb-95af-decad61f2998.png">
줄여놓은 MTU는 라우팅 경로가 변경될 경우, 작은 크기의 MTU로 계속 전송된다면 패킷의 낭비가 발생하게 되므로 다시 증가시켜야하는 경우도 발생한다.
PMTU 값의 증가는 주기적(2분 or 10분)으로 MTU 값을 1씩 증가시켜 전송하여 후에 이 크기가 계속 증가하여 상대방 라우터의 MTU 크기를 초과하였을 때,
**ICMP type3, code4** '```Fragmentation needed but DF bit set```'를 다시 수신하면 MTU 값 증가를 중단한다. <br/>
이후에는 수정된 MTU 크기로 ```DF = 1```을 설정하여 단편화 없이 통신을 할 수 있게 된다.

만약 방화벽이나 라우터 구간에서 ICMP 프로토콜을 차단했다면, 특정 장비로부터 전송된 **ICMP type3, code4** '```Fragmentation needed but DF bit set```' 패킷도 유실되고, 조정되지 않은 MTU와 DF=1로 설정된 TCP 패킷 전송 때문에
정상적인 TCP 통신이 되지 않는 상황이 발생할 수 있다. 이는 ICMP에 대해 무조건 차단했을 때 발생할 수 있으며 방화벽이나 라우터 구간에서 ICMP를 차단하였다면
'**ICMP type3 code4**'에 해당하는 '```Fragmentation needed but DF bit set```'을 허용하거나, MTU를 강제적으로 작은 쪽에 맞추는 방안을 사용해야 한다.

TCP를 이용한 DDoS 공격 발생 시, 만약 단편화된 TCP 패킷이 존재한다면 무조건 차단을 해도 무방하다.
라우터 구간에서 TCP 단편화 패킷을 차단해도 서비스에 지장이 없다.

### 🧸 Fragmentation Flooding 공격 패킷 분석
공격자는 주로 대역폭 공격을 발생할 때 의도적으로 큰 트래픽을 생성하기 위해 위조가 용이한 UDP 또는 ICMP 패킷을 MTU 이상의 큰 크기로 전송하며, 이때 단편화된 패킷들은
Flags 헤더가 '```More Fragments(MF)```'로 설정되어 전송된다.

<img width="1286" alt="스크린샷 2022-05-19 오후 4 17 36" src="https://user-images.githubusercontent.com/66156026/169234094-e60fdf6c-d45d-483c-9b97-8ab77199808a.png">

UDP Flooding 시에 데이터의 크기를 MTU 이상으로 전송하여 단편화된 공격 패킷으로 ```More Fragments```가 ```1```로 설정되어 있으며 데이터가 의미 없는 값으로 생성되어 있다.

<img width="1286" alt="스크린샷 2022-05-19 오후 4 24 00" src="https://user-images.githubusercontent.com/66156026/169235876-9489be00-0ad7-47f3-ae79-c0fb1d5b7ed3.png">

SYN Flooding은 본래 자원을 고갈시키기 위한 목적의 공격 유형이지만, 패킷의 크기를 크게 위조하여 발생할 때는 대역폭 공격과 같은 효과가 발생한다. 만약 MTU 이상의 크기로 전송하였다면 SYN 패킷은 단편화되어 전송된다.
<br/>다음 SYN Flooding 공격 패킷은 SYN 패킷의 크기가 MTU 이상의 크기로 전송되어 단편화되었고, 데이터는 의미없는 값으로 구성된 것을 확인할 수 있다.

<img width="730" alt="스크린샷 2022-05-19 오후 4 25 44" src="https://user-images.githubusercontent.com/66156026/169236263-91e6793d-a8c4-4175-9175-82dcc9f5b4ed.png">

실제 TCP 데이터가 전송될 때는 TCP 세그먼트가 IP 패킷으로 캡슐화되어 전송되며 단편화는 IP 패킷에 해당하는 Layer3에서 이루어진다.

### 🧸 Fragmentation Flooding 대응 방안
다른 대역폭 공격과 마찬가지로 회선 대역폭을 고갈시키는 대역폭 공격이므로, 대응 방안은 UDP Flooding과 비슷하다.
- 충분한 네트워크 대역폭 확보
- 미사용 프로토콜 필터링
- 위조된 IP 차단
- 출발지 IP별 임계치 기반 차단
- 패킷 크기 기반 차단
- 서버 대역폭 및 가용량 확대
- Anycast를 이용한 대응
- Ingress 필터링과 Egress 필터링

**Fragmentation 패킷 차단** <br/>
TCP의 경우 MSS와 PMTUD에 의해 MTU의 크기를 초과하지 않으므로 단편화될 가능성이 없고, UDP와 ICMP 또한 특수한 경우를 제외하고는 단편화될 만큼 큰 패킷으로 통신하는 경우가 거의 없으므로
자신이 운영하는 네트워크의 패킷을 조사한 후 단편화된 패킷이 전송되지 않는 것이 확인되면 UDP와 ICMP의 단편화 패킷도 차단하는 방법을 적용할 수 있다.

특수한 목적에 의해 단편화를 사용하는 네트워크 환경은 제외, 단편화 패킷 차단 설정 전, 자신의 네트워크에 단편화된 패킷이 통신되고 있는지 수일간의 모니터링 후 적용해야 한다.