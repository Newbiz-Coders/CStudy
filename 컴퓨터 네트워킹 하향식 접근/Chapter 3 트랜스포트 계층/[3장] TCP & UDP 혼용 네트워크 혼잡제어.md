# 사전지식

## TCP / UDP 핵심 요약

|  | TCP | UDP |
| --- | --- | --- |
| 사용 목적 | 신뢰성 있고 순서가 보장된 데이터 전송 | 빠른 데이터 전송 |
| 사용 예 | 이메일, 파일 전송, 웹 브라우징 | 실시간 게임 서버, 실시간 스트리밍 |
| 네트워크 혼잡 시 | 혼잡 제어를 통해 데이터 전송 속도 조절 | 혼잡 제어하지 않음 |
| 속도 | 비교적 느림 | 빠름 |

## 혼잡 윈도우(CWND)

- 혼잡 윈도우
    - TCP에서 사용하는 변수
    - 송신 가능한 데이터 크기를 **MSS(Maximum Segment Size)** 단위로 제한
- TCP의 혼잡 제어
    - 혼잡을 감지하면 혼잡 윈도우를 줄인다.
    - 이후 점진적으로 윈도우 크기를 다시 늘려가며 최적의 전송 속도를 찾는다.
- UDP의 특성
    - 혼잡 제어를 수행하지 않으며, 네트워크 상태와 관계없이 데이터를 계속 전송한다.
    - 이로 인해 혼잡 상황에서도 속도가 유지되지만 데이터 손실 가능성이 커질 수 있다.

# TCP & UDP 혼용 네트워크

## 게임 서버

하나의 게임 애플리케이션은 TCP와 UDP를 모두 사용한다.

예시:

- 실시간 캐릭터의 위치 정보 → **UDP**
    - 캐릭터의 위치 정보는 빠른 전송이 중요하다.
    - 약간의 데이터 손실은 게임 진행에 큰 영향을 주지 않는다.
- 게임 내 유저 간 귓속말 → **TCP**
    - 유저 간 메시지는 데이터 손실 없이 순서대로 전달하는 것이 중요하다.
    - 신뢰성이 핵심이다.

## 실시간 스트리밍

스트리밍 플랫폼도 TCP와 UDP를 혼용한다.

예시:

- 스트리머의 송출 영상 → **UDP**
    - 영상 데이터는 빠른 전송이 중요하며, 약간의 데이터 손실은 허용된다.
- 유저들의 채팅 → **TCP**
    - 채팅 메시지는 정확하고 순서대로 전달되는 것이 중요하다.

# 문제점과 해결책

## 문제점 → 네트워크 포화 상태에서 성능 저하

- TCP는 혼잡 제어 매커니즘을 통해 **전송 속도를 조절**
- UDP는 혼잡 제어를 수행하지 않고, 네트워크 상태 상관없이 **전송 속도 유지**
- 따라서, 다음 상황이 발생한다.
    1. UDP 트래픽 증가
    2. 네트워크 혼잡
    3. 혼잡을 감지한 TCP의 데이터 전송 속도 조절
    4. TCP 트래픽 감소
- **UDP의 트래픽이 증가했는데, TCP의 트래픽이 감소한다.**

## 해결책

### QoS(Quality of Service)

- 우선순위 기반의 자원 분배
    - 네트워크 자원을 트래픽의 중요도에 따라 할당하여 안정성과 성능을 보장.
    - TCP 트래픽에 높은 우선순위를 부여하여 신뢰성과 순서를 보장하는 데이터 전송이 중단되지 않도록 조치.
    - UDP 트래픽의 비율을 제한하여 네트워크 포화 상태를 방지.
- 반대로, UDP 트래픽에 높은 우선순위를 부여하기도 한다 (VoIP 통화)

---

### 트래픽 관리 모니터링

- 실시간 네트워크 트래픽 분석
    - 혼잡 상황에서 트래픽 유형을 모니터링하여 UDP 트래픽이 비정상적으로 증가하는 경우 이를 자동으로 감지 및 제어.
    - 네트워크 대역폭이 불균형적으로 사용되지 않도록 지속적으로 관리.
- DPI(Deep Packet Inspection) 기술 활용
    - DPI는 패킷의 내용을 분석하여 트래픽의 성격(예: 웹 브라우징, 스트리밍, 파일 전송 등)을 구분하는 기술.
        - 비인가된 UDP 트래픽은 제한하거나 차단.
        - TCP 트래픽이 필요한 자원을 확보할 수 있도록 제어.
    - 패킷의 내용을 검사 → 보안 및 개인정보 보호 측면 에서 윤리적 고려 필요

---

### QUIC

- QUIC는 UDP 기반에서 작동하면서, TCP의 안정성과 신뢰성을 통합한 프로토콜
- UDP 에서 수행하지 않던 혼잡 제어 기능이 추가됨
    - TCP와 동일하게 네트워크 혼잡을 감지하고 전송속도를 줄여, TCP와 공정한 대역폭 분배 가능

# 공평성이 꼭 필요할까?

- **상황에 따라 다르다.**
    - 네트워크 자원의 공평한 분배는 항상 필요한 것이 아니라, 애플리케이션의 특성과 서비스의 우선순위에 따라 달라질 수 있다.

## 공평성이 필요하지 않은 경우

### 애플리케이션 특성에 따라

- 실시간 정보가 중요한 게임/스트리밍의 경우 메시지 기능과 같은 TCP 트래픽의 성능 저하를 감수 하더라도, UDP로 보내는 실시간 데이터가 더 중요

---

### 상황에 따른 적응적 관리

- VoIP 통화와 파일 다운로드가 동시에 발생할때. 파일의 다운로드가 조금 느려져도 끊김 없는 통화가 우선시 됨.

---

## 공평성이 필요한 경우

### 필수 서비스 보호

- 과도한 UDP 트래픽으로 인해 금융 거래, 의료 데이터 등의 TCP 트래픽의 전송 실패가 발생하면 사용자와 서비스 제공자 모두에게 큰 손실 초래

---

### 공용 네트워크의 공정한 사용

- 공용 Wi-Fi 환경에서 다수의 사용자가 동일한 네트워크를 사용하는 경우, 대규모 스트리밍과 같은 특정 트래픽이 과도하게 대역폭을 차지할경우 다른 사용자들이 기본적인 인터넷 서비스를 이용할 수 없음
