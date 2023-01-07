가시화는 그래프 형태로 시각적으로 인지하기 쉬운 자료로 가공하기 위함으로 이를 통해 분석법 또는 절차에 대한 빠른 판단, 도출된 자료를 바탕으로 어떠한 의사결정을 하는데 도움을 줄 수 있다.

### 💡 Gnuplot
Gnuplot을 활용하면 패킷 덤프 내 원하는 프로토콜에 대한 통계 데이터를 추출하여 그래프 이미지를 만들 수 있다.
2차원, 3차원 함수, 자료를 그래프로 그려주는 명령행 응용 소프트웨어이다.

먼저 tcpstat를 활용하여 데이터를 추출하고, gnuplot.script를 작성하여 그래프 이미지를 추출한다.

tcpstat downloads ```$ brew install tcpstat``` <br/>
gnuplot downloads ```$ brew install gnuplot```

```
tcpstat -r <file name> -o "%R₩t%A₩n" 60 > arp.data
tcpstat -r <file name> -o "%R₩t%C₩n" 60 > icmp.data
tcpstat -r <file name> -o "%R₩t%T₩n" 60 > tcp.data
tcpstat -r <file name> -o "%R₩t%U₩n" 60 > udp.data
```

옵션|설명
:---:|:---:
```-r```|데이터를 읽어들일 파일을 지정. pcap, snoop 등의 파일 포멧을 지원한다.
```-o```|통계를 출력할 때 표시할 형식을 지정하는 옵션.
```%R```|첫번째 패킷의 상대시간 값
```%A,C,T,U```|차례대로 ARP, ICMP, TCP, UDP를 의미한다.
```\t \n```|\t: 탭 출력, \n: 개행 출력

60이라는 숫자의 의미는 첫 번째 시간 값을 60초 단위로 개행하며 기록한다는 것이다.

아래는 위에서 추출한 프로토콜의 패킷 통계 그래프 이미지를 생성하는 예제 스크립트이다.

```
# file name: proto.script
set term png small
set style data lines
set grid
set yrange [ -10 : ]
set title "protocol statistics"
set xlabel "seconds"
set ylabel "packets/min"
plot	"arp.data" using 1:2 smooth csplines title "ARP" \
, "icmp.data" using 1:2 smooth csplines title "ICMP" \
, "tcp.data" using 1:2 smooth csplines title "TCP" \
, "udp.data" using 1:2 smooth csplines title "UDP"
```

- set은 그리드에 대한 설정과 출력할 이미지에 표시될 타이틀, 스타일과 이미지 크기, X/Y축 라벨명과 min, max 값을 정의한다.
- plot은 사용할 데이터 파일명, 비율 표시 데이터 형식, 각 데이터의 라벨명을 정의한다.

```$ gnuplot proto.script > protocol.png```

❌ **오류 내용** <br>

![error](https://user-images.githubusercontent.com/66156026/211141334-d9cd3872-fe23-41ec-872d-b0087c411ac5.png)

통계 그래프는 Wireshark [Statistics] - [I/O Graph] 에서도 추출이 가능하다.

<br/>

### 💡 Graphviz, AfterGlow
출발지, 목적지 IP/PORT를 기준으로 노드 간의 방향성을 가시화하는 방법이다. 주로 1:N 형태의 스캔성 트래픽, DDoS와 같은 분산 집중 형태의 N:1 형태의 공격을 빠르게 파악하는 데 도움이 된다.

Graphviz는 간단한 문법의 텍스트를 분석하여 자동으로 그래프를 만들어 주는 도구이다. 여기서 제공하는 도구 중 가장 유용하게 쓰이는 것은 **neato**와 **dot**이다. 
dot은 DAG(directed acyclic graph)를 생성할 때 주로 사용하고 neato는 일반적인(undirected) 그래프를 생성할 때 사용한다. 
그래프는 bmp, gif, png, svg, ps, pdf 등의 다양한 형식을 출력할 수 있다.

순서는 tcpdump를 이용해 출발지/목적지 IP와 목적지 PORT를 추출하고 AfterGlow perl 스크립트를 사용하여 neato에서 인식할 수 있는 자료로 출력한다.
그 후 neato에서 gif로 노드 간 트래픽의 흐름을 추출한다.

Graphviz downloads
```
$ brew install graphviz
$ pip install graphviz
``` 
<br/>
AfterGlow downloads https://github.com/zrlram/afterglow

```$ tcpdump -vttttnnelr <file name> | perl ./parsers/tcpdump2csv.pl "sip dip dport" | perl ./afterglow_sample/afterglow.pl -c ./parsers/color.properties | neato -Tgif -o ./test.gif```

1. tcpdump로 파일에서 필요한 데이터를 추출하고
2. AfterGlow에서 제공하는 tcpdump2csv.pl 스크립트를 사용하여 출발지/목적지 IP, PORT를 출력한다.
3. 출력된 내용을 AfterGlow.pl 스크립트를 이용하여 데이터를 파싱하고
4. 파싱된 데이터를 Graphviz의 neato에서 이미지 그래프 파일로 저장한다.
5. (옵션으로 AfterGlow의 color.properties를 사용하였는데, 원하는 색상으로 변경이 가능하다.)

