---
layout: single
title:  "TCP, UDP, handshaking "
categories: Network
tag: [network, 네트워크, cs]
toc: true
author_profile: true
---


## TCP

- reliable, in-order byte stream (←→ udp : unreliable, message boundaries)
- **connection** oriented ⇒ **handshaking**
- 같은 connection 위에서 **양방향 통신** (반면, HTTP는 단방향 통신임)
- +) **pipelined** : ACK을 받지 않고도 연속으로 packet을 보내는 것. 이때 연속으로 보내는 packet의 개수 ⇒ window size

## TCP vs UDP

![image](https://user-images.githubusercontent.com/47748246/192450690-45ea1427-57b0-4a29-bc1e-ef3bb243311b.png)

- UDP
    - 각 메시지가 독립적 (not 연속적) → 서버는 소켓 하나로 여러 클라이언트와 통신할 수 있다. 
- TCP
    - 메시지가 byte-stream 형태 → 클라이언트 요청 별로 서버에서 소켓을 생성하는 이유 (각 클라이언트 별로 보내는 연속적인 메시지들 -> 어떤 클라이언트가 어떤 메시지 다음으로 보냈는지 구별하기 위해)

![image](https://user-images.githubusercontent.com/47748246/192450744-41e6cfaf-5655-4efc-9a03-28dc4d19b5b8.png)

## TCP 3 way handshaking과 4 way handshaking

### TCP 3-way-Handshaking

- 양 쪽 모두 **데이터를 전송**할 준비가 되었다는 것을 보장

![image](https://user-images.githubusercontent.com/47748246/192450776-b04711e2-78d5-4675-b76b-749cd38b09e8.png)

1. 클라이언트 → 서버 : 접속을 요청하는 SYN 패킷을 보냄. 이때 클라이언트는 SYN_SENT 상태 (SYN/ACK 응답을 기다리는 상태)
2. 서버 → 클라이언트 : SYN 요청을 받고 요청을 수락하는 ACK와 SYN falg가 설정된 패킷을 발송. 이때 서버는 SYN_RECEIVED 상태
3. 클라이언트 → 서버 : ACK을 보내고 TCP 연결이 완성. → 데이터가 오가게 됨

### TCP 4-way-Handshaking

- 세션을 종료하기 위해 사용됨

![image](https://user-images.githubusercontent.com/47748246/192450812-909cc286-219d-4ba4-85c6-09dbd71531ff.png)

1. 클라이언트 → 서버 : 연결을 종료하겠다는 FIN 플래그 전송
2. 서버 → 클라이언트 : 일단 확인 메시지를 보내고 자신의 통신이 끝날때까지 기다림. (TIME_WAIT 상태)
3. 서버 → 클라이언트 : 통신이 끝났으면 연결이 종료되었다고 클라이언트에게 FIN 플래그 전송
4. 클라이언트 → 서버 : 확인했다는 메시지 전송
