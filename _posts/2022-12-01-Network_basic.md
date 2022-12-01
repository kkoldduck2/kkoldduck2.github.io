---
layout: single
title:  "네트워크 기초 이론"
categories: Network
tag: [network, 네트워크, cs]
toc: true
author_profile: true
---
### 1. 네트워크 계층 : 이론과 구현

![image](https://user-images.githubusercontent.com/47748246/204977329-a5acd420-3d5d-48f3-8c81-7611f7518d86.png)

- 커널의 구성요소(프로토콜)를 유저모드 어플리케이션으로 추상화 ⇒ File 형태 (socket이라고 불린다)
- **Socket** : TCP를 user mode application이 접근할 수 있도록 File 형식으로 추상화한 인터페이스
- OSI 7 layer보다는 “구현”에 집중할 것 → HTTP, TCP/IP를 중심으로 공부해라

### 2. Host, Switch, Network 간의 관계

![image](https://user-images.githubusercontent.com/47748246/204977351-a81476fe-7a09-408d-9e60-0754423afe97.png)
- Host = Network에 연결된 컴퓨터
    - Net 이용 주체로서 host ⇒ End-point
    - Net 자체로서 host  ⇒ Switch
        - Router → 경로 찾기
        - Fire wall, IPS → 보안
- Switch
    - L2 계층에서 mac address로 스위칭 → L2 Switch
    - L3 계층에서 IP address로 스위칭 → L3 Switch
    - L4 계층에서 port 번호로 스위칭 → L4 Switch
    - L7 계층에서 프로토콜(HTTP)의 뭐로 스위칭 → L7 Switch
- Network → **`Internet = Router + DNS`**

### 3. IPv4 주소 체계에 대한 암기사항

**1) IP 주소란?**

![image](https://user-images.githubusercontent.com/47748246/204977382-dd17a11e-8d5a-4dc8-981e-ee9148e35696.png)

- IP 주소 : Host에 대한 식별자
- 대한민국의 행정체계를 Network라고 한다면, 그 안에 속해있는 개인을 식별하는 주민번호를 IP 주소라고 할 수 있다.
- IP 주소는 IPv4와 IPv6 2가지 타입이 존재한다.
    - IPv4 : 주소 길이 = 32bit → 2^32 (43억개 가능)
    - IPv6 : 주소 길이 = 128bit

**2) IPv4 주소 체계의 구성 & SubnetMask**

![image](https://user-images.githubusercontent.com/47748246/204977428-274b622a-da80-4dec-8c2a-dbd6ac58f241.png)

- IP 주소에서 Network ID의 길이를 나타내는 것 = NetMask (SubnetMask)
- IP 주소는 .을 사이에 두고 8bit씩 끊어서 표시한다. 그리고 그 단편은 0~255까지 표현 가능하다.

![image](https://user-images.githubusercontent.com/47748246/204977538-452f6845-71ae-4d75-9060-812cadacf2b6.png)

- Host의 IP 주소 (192.168.60.14)와 서브넷 마스크(255.255.255.0)를 and 연산 ⇒ Network ID
- 192.168.60.14/24 : host ip는 192.168.60.14이고, 이 중에서 서브넷 마스크 길이는 24bit이다.

### 4. Port 번호 이해

![image](https://user-images.githubusercontent.com/47748246/204977519-c9146b78-db19-424e-98a1-f5c48c5e3443.png)
- TCP/IP가 kernel 수준에서 implementation이 되어있음. 이걸 user mode application이 접근할 수 있도록 **`인터페이스`**가 제공됨 → 이 인터페이스의 본질은 file이지만, **프로토콜을 추상화**했기 때문에 이 File을 **`socket`**이라고 부른다.
- 이 Socket에 attach되는 정보 중 하나가 Port 번호이다.
- Port 정보는 기본적으로 16bit 정보이다.

### 5. Switch가 하는 일은 Switching이다.

![image](https://user-images.githubusercontent.com/47748246/204977578-1980aeda-66a5-4cd8-a61e-c477c4cef02b.png)

- switching : 경로 선택 or 인터페이스 선택
- 우리가 교차로에서 어느 경로로 갈지 선택할 때, 그 근거로 이정표를 본다.
- **Internet은 Router의 집합체**라고 볼 수 있다. **Router는 기본적으로 L3 스위칭을 지원**한다.
- 네트워크를 고속도로라고 했을 때, 교차로를 Router라고 할 수 있다.
- 위 그림에서 교차로 A, B, C..는 **네트워크 인터페이스가 4개인 라우터**이다.
- 라우터는 들어온 인터페이스를 제외한 3개의 인터페이스 중 어느 경로를 선택해야 최적일 지를 결정 (라우터들끼리 통신해서 알아낸 정보를 기반으로 결정)
- 위 그림에서 자동차는 Packet이다. 즉, packet 단위의 데이터가 라우터에 도착하면 Switching(경로 선택)을 한다. 그 중 최적화된 경로를 통해서 목적지에 도달하는데, 그때 근거가 되는게 **라우팅 테이블(이정표)**이다.

### 6. 네트워크 데이터 단위 정리

![image](https://user-images.githubusercontent.com/47748246/204977614-0553d63e-0193-4fcf-a156-b81bf1c0c2b2.png)

- 복습) 커널을 추상화한 인터페이스가 File인데, 이를 Socket이라 한다.
- File의 데이터 단위 : **`Stream`**
    - Stream 특징 : 데이터 길이가 어떻게 될지 모른다. 계속 길어질 수 있다.
- TCP의 데이터 단위 : **`Segment`**
- IP의 데이터 단위 : **`Packet`**
- Access 계층에서 다루는 데이터 단위 : **`Frame`**

1. 어플리케이션 프로세스가 File(socket)에 데이터를 Write (Stream) 
    - 즉, 어플리케이션 프로세스 수준(=Socket 수준)에서는 Stream 데이터이다.
    - Stream 데이터는 그 끝을 알 수 없는 일렬로 쭉 늘어진 데이터이다.
2. 이 데이터 Stream이 유저모드에서 커널 모드로 넘어가면서 TCP를 만나면, **Stream → Segment로 자르기(Segmentation)**가일어난다. 
3. 이 Segment가 IP 프로토콜에서 다시 Packet으로 encapsulation되고, 
4. 이 Packet이 Access 계층에서 다시 Frame으로 encapsulate된다. 

### 7. 네트워크 인터페이스 선택 원리와 기준

![image](https://user-images.githubusercontent.com/47748246/204977638-0b35b3e2-e829-426d-887a-13a60866d114.png)

- NIC (Network Interface Card)가 2개이면 통상적으로 IP 주소도 2개가 생긴다.  (IP주소는 항상 NIC와 연결된다.)
- 메트릭 : 해당 IP를 선택 (= NIC)를 선택했을 때 비용 → 이걸 기준으로 어떤 NIC를 사용할지 결정한다.

### Reference
[https://www.youtube.com/watch?v=k1gyh9BlOT8&list=PLXvgR_grOs1BFH-TuqFsfHqbh-gpMbFoy](https://www.youtube.com/watch?v=k1gyh9BlOT8&list=PLXvgR_grOs1BFH-TuqFsfHqbh-gpMbFoy)
