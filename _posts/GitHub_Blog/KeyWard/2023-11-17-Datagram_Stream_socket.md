---
title: "Datagram socket, Stream socket"
last_modified_at: "2023-11-17T22:00:00"
categories:
  - 크래프톤 정글
  - CS
  - 네트워크
tags:
  - 크래프톤 정글
  - CS
  - 네트워크
  - 소켓
---

## Datagram Socket
 비연결형(Connectionless) 소켓이며,<br>
 UDP(User Datagram Protocol) 기반<br>
 (UDP는 Transfort Layer의 프로토콜이며,<br>
 한 계층 아래의 IP 프로토콜과 유사한 특징을 가진다)<br>

 비신뢰성이 존재하기에<br>
 데이터가 손실될 수 있고, 순서가 보장되지 않을 수 있음<br>

 데이터그램은 '패킷' 단위로 전송되며, 데이터의 경계를 유지<br>

 각 데이터그램은 독립적으로 처리되며, 수신 순서와 전송 순서가 일치하지 않을 수 있음<br>

 UDP는 연결 설정 과정이 없으며, 데이터를 보내고 받을 때마다<br>
 목적지 IP와 포트를 명시하여 데이터를 전송한다<br>

 UDP는 보통 패킷을 분할하지 않으나<br>
 데이터가 아주 큰 경우(MTU : Maximum Transmission Unit 이상)<br>
 패킷으로 분할되어 전송할 수 있음<br>
 (보통 65,507 바이트로 제한이며, 이보다 작아야 안정적)<br>
 (실제로 이렇게 최대 크기로 보내는 것은 드뭄)<br>
 (이더넷 프레임에서 1500바이트 로 제한되기에...)<br>

 보통 이러한 경우는 데이터가 손실될 수 있으며,<br>
 전송 순서가 보장되지 않기에 추가적인 구현이 필요함<br>

 UDP의 순서가 중요한 경우는<br>
 Application Layer에서 데이터 손실 혹은 순서 문제에 대한 처리를 구현해야 함<br> 

 checksum 기능은 존재<br>
 -> 보내진 정보가 잘못되었는지 여부는 확인해야 하므로<br>

 IP와 같이 'best-effort' 서비스를 제공<br>
 최선을 다하지만, 품질 보장 없음, 신뢰성 낮음....<br>
 (최선을 다한다? - 최적 경로 및 방법)<br>


## Stream Socket
 연결형(Connection-oriented) 소켓이며,<br>
 TCP(Transmission Control Protocol) 프로토콜을 기반으로 작동<br>
 
 신뢰성이 높기에<br>
 데이터가 손실되거나 순서가 바뀌지 않음<br>

 연결 설정, 통신 종료를 위한 handshake 과정이 있어<br>
 신뢰성과 안정성이 높음<br>

 sequence 번호(그리고 ACK 번호)는 전송되는 데이터를 byte 단위로 트래킹<br>

 IP packet이 헤더 포함 64kb까지 이므로<br>
 TCP는 전송하는 데이터 양에 따라 IP packet 여러 개로 쪼개서 전송<br>

 수신 측에서는 여러 IP에 걸쳐있는 TCP segment를<br>
 sequence 번호를 이용해 결합해서 복원한다<br>

 이 때문에 TCP는 하나씩 끊겨서 전송되는 IP 위에서<br>
 마치 data가 연속으로 흐르는 것 같은 (stream) 환경을 지원<br>

 TCP는 특정 상대방을 가정하고 전송한 data의<br>
 byte 단위 sequence 번호, 어디까지 수신했는지 byte 단위의 ack 번호를 관리<br>

 통신 양 끝단(end-point)의 state를 관리하기에<br>
 stateful 하고 connection 기반<br>

 - 전송 제어<br>
   전송 제어중 신경쓰는 요소는<br>
   수신자의 여력(버퍼 크기) -> TCP header의 Window Size<br>
   중간에 거쳐가는 게이트웨이(라우터)의 여력(버퍼 크기) -> 받았는지 여부(TCP header의 SEQ/ACK)<br>

 - TCP Window size<br>
   상대방에게 자신이 얼마나 큰 데이터를 한 번에 받을 수 있는지 advertise 한다<br>
   (receive window size)<br>

   socket library로는 setsockopt 로 변경 가능<br>

   그러나 여기서 advertise 된 window size 단위로 데이터가 전송되지 않고,<br>
   혼잡 제어 알고리즘에 다른 window 크기와 비교하여 '작은 사이즈'로 전송<br>