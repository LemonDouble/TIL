# HTTP : 인터넷 통신

배운 내용 (한줄 요약): IP, TCP, UDP, PORT, DNS
분류: HTTP
작성일시: 2021년 7월 19일 오후 5:18

(간단하게 정리함)

## 1. OSI / Internet Layer

- OSI 7계층
    1. Application Layer
    2. Presentation Layer
    3. Session Layer
    4. Transport Layer
    5. Network Layer
    6. Data Link Layer
    7. Physical Layer 

- Internet Layer
    1. Application Layer ( IMAP,SMTP, FTP, HTTP... )
    2. Transport Layer (TCP, UDP..)
    3. Internet Layer (IP, ARP, ...) - IP Addr
    4. Data Link Layer (Ethernet , 802.11(wifi) ...) - MAC Addr
    5. Physical Layer : (Coax, fiber ...) 

## 2. IP (인터넷 프로토콜)

- 지정한 IP 주소 (IP Address) 에 데이터 전달
- 패킷 (Packet) 이란 단위로 데이터 전달

- IP Packet
    - Src IP + Dest IP + Payload

- IP 프로토콜의 한계
    1. 비연결성 : 패킷을 받을 대상이 없거나, 서비스 불능 상태여도 패킷 전송
    2. 비신뢰성 : 패킷이 사라지거나, 패킷이 순서대로 오지 않을 수 없음
    3. 프로그램 구분 불가 : 같은 IP에서 어플리케이션이 2개 이상이라면? 

## 3. TCP, UDP

- TCP Packet
    - Src Port # +  Dest Port # + 전송 제어 + 순서 + Validation..

- TCP의 특징 : 전송 제어 프로토콜 (Transmission Control Protocol)
    - 연결 지향 - TCP 3 way handshake (가상 연결)
    - 데이터 전달 보증
    - 순서 보장
    - Reliable Protocol

- 연결 지향 - TCP 3-way handshake
    1. TCP SYN (SYN flag = 1)
    2. TCP SYN + ACK (SYN flag = 1, ACK flag = 1)
    3. ACK (ACK flag = 1)
    4. 데이터 전송

- UDP : 사용자 데이터그램 프로토콜 (User Datagram Protocol)
    - 데이터 전달/순서가 보장되진 않지만, 단순하고 빠름
    - Port + Checksum 정도만 추가로 제공
    - Applicaion에서 추가 작업 필요

## 4. Port

- 같은 IP 내에서 Process를 구분하는 구분자
- 0~65535 : 할당 가능
- 0~1023 : Well-known port : 사용하지 않는 것이 좋음
    - FTP : 20,21
    - TELNET : 23
    - HTTP : 80
    - HTTPS: 443

## 5. DNS

- DNS : Domain Name System
- 도메인 명을 IP 주소로 변환