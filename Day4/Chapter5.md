# 5. 인프라를 지탱하는 응용 이론

## 5.1 캐시

### 5.1.1 캐시란?

사용 빈도가 높은 데이터를 고속으로 액세스 할 수 있는 위치에 두는 것

- CPU 1차, 2차 캐시
- 저장소 캐시
- OS 페이지 캐시
- 데이터베이스 버퍼 캐시
- KVS (데이터를 메모리에 캐시하는 것)

임시 저장소를 의미하기도 함



### 5.1.2 어디에 사용되나?

#### 브라우저 캐시

웹 서버에서 매번 콘텐츠를 다운로드하지 않고, 브라우저에 접속한 페이지의 콘텐츠를 캐싱

#### CDN (Content Delivery Network)

웹 서버와 클라이언트 사이에 배치하는 캐시 서버 (또는 네트워크)

아카마이는 처음 들어보고 **짱짱 AWS** **CloudFront**



### 5.1.3 정리

#### 장점

- 데이터 고속 액세스
- 실제 데이터의 부하를 줄임
    - 참조 빈도가 높은, 읽기 전용 데이터에 적합

#### 단점

- 데이터를 잃을 위험이 있음 (쓰기에 사용할 경우)
    - 데이터 갱신 빈도가 높은 시스템, 대량의 데이터의 경우 부적합



## 5.2 끼어들기 (Interrupt)

### 5.2.1 인터럽트란? & 5.2.2 자세히 보자 & 5.2.3 어디에 사용되나?

> 이 섹션의 모든 내용들은 운영체제 책에서 배울수 있씁니다!

- 애플리케이션 실행 중 키보드 입력 시
  - IO Interrupt발생, CPU가 처리하던 프로세스 일시 중지
  - 키보드 Input실행
  - 다시 프로세스로 복귀

- 웹 사이트 접속 시
  - 서버 [NIC(Network Interface Controller)](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4_%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC)에 [이더넷 프레임](http://www.ktword.co.kr/abbr_view.php?m_temp1=2965)이 도착
  - NIC가 CPU에 Interrupt발생시킴
  - CPU는 작업중이던 프로세스 정보를 메모리에 저장
  - CPU를 사용해 NIC의 데이터 수신
  - 다시 메모리에 저장된 정보를 가져와 프로세스 처리 재개

- 프로세스 또는 쓰레드가 허가되지 않은 메모리 위치에 액세스 하려고 할 시
  - [Segmentation Fault](https://ko.wikipedia.org/wiki/%EC%84%B8%EA%B7%B8%EB%A9%98%ED%85%8C%EC%9D%B4%EC%85%98_%EC%98%A4%EB%A5%98)가 발생, OS가 Interrupt를 발생시켜 프로세스 강제 종료

### 5.2.4 정리

- 인터럽트는 어떤 일이 발생하면 연락하는 '이벤트 주도' 구조
- CPU가 정기적으로 상태를 체크하는 방법은 폴링(polling)
- 폴링 간격을 정확히 잡을 수 없기 때문에 인터럽트가 더 효율적



## 5.3 폴링

### 5.3.1 폴링이란?

- 특징
  - 정기적 질의
  - 단방향 요청
  - 일정 간격으로 정기적으로 발생
- 장점
  - 루프형태여서 프로그래밍이 쉬움
  - 상대 응답을 확인 가능
  - 일정 데이터를 모아서 처리 가능

### 5.3.2 어디에 사용되나?

- 웹 로직 - 서버의 접속 감시
  - Application서버 - DB서버간의 연결을 생성
  - App서버에서 정기적으로 DB서버와의 연결을 테스트
- [NTP(Network Time Protocol, 시간 동기)](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%83%80%EC%9E%84_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C) 처리
  - 서버 - NTP서버
  - 서버에서 [NTPD](https://en.wikipedia.org/wiki/Ntpd)프로세스가 주기적으로 NTP서버와의 통신으로 시간을 동기화

### 5.3.3 정리

- 적합
  - 일정 간격으로 처리하면 좋은 경우
  - 감시
- 부적합
  - 상태가 아닌 입력 내용으로 실행내용을 변경하는 처리
  - 처리 우선순위를 정해야 하는 경우
- 주의
  - 네트워크를 경유한 폴링은 간격이 너무 짧을 경우, 트래픽 및 서버의 리소스 소비가 증가한다



## 5.4 핑퐁

### 5.4.1 핑퐁이란?

- 인프라 설계나 성능 튜닝에 있어 중요한 개념
- 상자가 너무 작아 짐을 운반하기 위해 몇 번이고 왕복해야 하는 상황을 '핑퐁'이라 칭함

### 5.4.2 어디에 사용되나?

- DB의 경우
  - 데이터 블록
    - DB가 파일을 읽기/쓰기하는 최소 단위
    - 블록이 8KB면, 1바이트를 읽을때도 8KB를 써야 함
    - 블록이 32KB면, 역시 1바이트를 읽을때도 32KB를 써야 함
  - 디스크 섹터 및 파일 시스템 블록
    - 데이터 블록보다 하위 계층의 읽기/쓰기 단위
    - 효율적인 블록 크기를 설정하기 위해서는 하위 계층의 배수를 사용해야 함
- 네트워크의 경우
  - 브라우저를 통해 데이터가 전달 될 때는 송신버퍼로부터 [TCP](https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1_%EC%A0%9C%EC%96%B4_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C) 세그먼트단위로 전송되며, 이 세그먼트는 [MSS (Maximum Segment Size)](https://en.wikipedia.org/wiki/Maximum_segment_size) 를 초과하지 않는 범위로 분할된다.
  - IP소켓의 최대 크기는 [MTU(Maximum Transfer Unit)](https://ko.wikipedia.org/wiki/%EC%B5%9C%EB%8C%80_%EC%A0%84%EC%86%A1_%EB%8B%A8%EC%9C%84) 이라 한다.
    - [OSI모형](https://ko.wikipedia.org/wiki/OSI_%EB%AA%A8%ED%98%95)
      - MSS는 애플리케이션 계층의 최대 사이즈
      - MTU는 네트워크 계층의 최대 사이즈
        - MSS는 MTU에서 네트워크 계층 헤더, 전송 계층 헤더를 제외한 사이즈를 최대 데이터 크기로 갖는다
  - 데이터 고속 전송 시스템에서는 소켓 버퍼나 MTU크기를 튜닝하기도 함
  - 데이터 전송 중, 경로의 라우터 MTU크기가 작다면 패킷이 분할되어야 하므로 오버헤드가 발생하고, 성능저하가 발생
    - 통신 경로에 따라 MTU크기를 적절히 설정해야 한다

### 5.4.3 정리

- 큰 상자는 대량의 데이터를 빠르게 운반 (처리량 중시)
- 작은 상자는 소량의 데이터의 경우 더 빠르게 운반 (지연 시간 중시)
- DB의 경우 상황에 따라 다른 읽기 방식이 존재



## 5.5 저널링

### 5.5.1 저널링이란?

- 트랜잭션이나 매일 갱신되는 데이터의 변경 이력
- 저널을 남겨두는 것을 저널링이라 함
- [저널링 파일 시스템](https://ko.wikipedia.org/wiki/%EC%A0%80%EB%84%90%EB%A7%81_%ED%8C%8C%EC%9D%BC_%EC%8B%9C%EC%8A%A4%ED%85%9C)
  - 파일 시스템에 변경사항을 반영하기 전에, 변경사항을 추적하는 파일 시스템
  - 중도에 오류 발생 시 기록된 변경사항을 사용해 복구
- 특징
  - 데이터 자체가 아닌 처리(트랜잭션) 내용을 기록
  - 데이터 일관성이나 일치성이 확보되면 필요 없음 (즉, 이 부분이 필요하므로 존재)
  - 데이터 복구시 롤백이나 롤포워드에 사용

### 5.5.2 어디에 사용되나?

- 데이터를 처리하는 기능의 대부분은 저널링이 구현되어 있음
  - ext3 파일 시스템
    - DB와는 달리 트랜잭션(작업)시, 버퍼 정보를 디스크에 기록하지 않음
    - 버퍼의 데이터를 잃을 수 있음
    - 기본은 5초에 한 번 기록
  - 오라클 DB
    - Redo로그라 불림
    - 트랜잭션 종료 시 버퍼가 디스크에 기록
    - 기록중인 Redo로그가 파손된 경우에는 데이터 복원 불가능
      - Redo로그를 이중화해서 보호

### 5.5.3 정리

- 장점
  - 시스템 장애 시 복구 빠름
  - 데이터 복제보다 적은 리소스로 데이터를 보호
- 적합한 경우
  - 데이터 갱신이 발생하는 시스템
- 부적합한 경우
  - 데이터 안정성보다는 성능을 요구하는 경우
    - 저널링은 기록 처리 시 오버헤드가 발생
    - 캐시 서버 등 실제 데이터가 다른곳에 있다면 불필요
- 복구
  - 체크포인트(데이터가 디스크에 기록된 것을 보증하는 지점)를 기준으로
    - 트랜잭션이 체크포인트 전에 시작되어 체크포인트 전에 종료된 경우
      - 정상작업이므로 복구 불필요
    - 트랜잭션이 체크포인트 전에 시작되어 장애발생전에 종료된 경우
    - 트랜재션이 체크포인트 후에 시작되어 장애발새전에 종료된 경우
      - 저널을 읽어 복구 (롤 포워드)
    - 트랜잭션이 체크포인트 전에 시작되어 장애발생까지 종료되지 않은 경우
      - 롤백으로 작업 변경내역을 되돌림 (복구하지 못함)
    - 트랜잭션이 체크포인트 후에 시작되어 장애발생까지 종료되지 않은 경우
      - 체크포인트 때의 데이터 상태로 유지됨 (복구하지 못함)
  - 저널 데이터는 버퍼에 저장되며, 기록 빈도가 많을 수록 복구 가능한 시점도 많아지지만 오버헤드도 높아짐
  - 트랜잭션 단위로 일치성을 보증하므로 트랜잭션 단위가 크다면 복구하지 못하는 데이터가 많아질 가능성이 커짐
- Copy-On-Write (COW)
  - 저널링은 데이터와 증감 정보를 나눔
  - COW는 모든 데이터를 새로 신규영역에 복사 후 완료되면 참조 위치 주소를 변경
  - 데이터 변경 중 장애가 발생해도 갱신중이었다는 정보가 남지 않음 (All or Nothing)
    - 단점은 최대 2배의 데이터 영역이 필요
    - 용량이 충분하다면 크게 문제되지 않을 수 있음

## 5.6 복제

### 5.6.1 복제란?

- 복제(replication), DB나 저장소 등에 자주 사용되는 기술
  - 예비 데이터 센터
  - 대량의 사용자 접속에 대비해 동일 데이터를 여러 서버에 복제, 부하 분산
- 특징
  - 장애 시 데이터 손실 예방
  - 복제를 이용한 부하 분산
- 장점
  - 사용자가 데이터 액세스 시 복제한 것인지 의식할 필요가 없음
  - 백업과 달리 실제 데이터가 복제 데이터와 실시간으로 동기화

### 5.6.2 어디에 사용되나?

- 원본 데이터 서버와 복제 데이터 서버
  - 원본 데이터 서버에서의 증감 데이터만 복제 위치로 반영
    - 데이터 전송량을 줄임
  - 데이터 보호를 최우선시, 쓰기 처리 시 데이터가 복제 될 때 까지 기다리도록 함 (동기)
- MySQL 복제
  - 데이터 '추가, 갱신, 삭제'등 변경 처리(트랜잭션)을 복제측으로 보냄
    - 데이터 블록이 아닌 증감 데이터만 보내므로 전송량을 줄임

### 5.6.3 정리

- 적합한 경우
  - 데이터 손실을 허용하지 않으며 장애 시 복구 속도가 빨라야 하는 시스템
  - 데이터 참조와 갱신이 나누어져 있으며, 참조가 많은 시스템
    - 복제 위치는 참조용으로만 사용, 부하 분산 및 확장성 향상
- 부적합한 경우
  - 데이터 갱신이 많은 시스템
    - 원본 데이터가 복제 데이터로 반영되어야 하므로 오버헤드가 높아짐
    - 경합 위험을 줄이기 위해 갱신은 한 곳에서 하는 것이 바람직
      - 갱신 처리의 부하분산에는 적합하지 않음



## 5.7 마스터-슬레이브

### 5.7.1 마스터-슬레이브란?

- 마스터-슬레이브
  - 상호 접속 관계
  - 한 사람이 관리자가 돼서 모든 리소스를 제어
    - 슬레이브 역할은 마스터로부터 리소스를 허가 받음
- 피어-투-피어
  - 구성원들이 같은 권한을 가짐
  - 각자 자신의 리소스를 사용

### 5.7.2 어디에 사용되나?

- 복제가 좋은 예가 됨
  - 실제 데이터는 마스터
  - 언제 슬레이브에 전파할지는 마스터가 결정함
    - 여러개의 슬레이브가 존재하면, 마스터 측 부하가 높아짐
- 현실에서는 마스터-슬레이브와 피어-투-피어의 장점만 조합해서 이용
  - 오라클 DB의 마스터-슬레이브 구성
    - 여러 물리서버들이 리소스 단위로 관리 마스터를 담당
    - 한 물리서버가 다운될 경우, 해당 서버가 마스터였던 리소스는 다른 마스터 서버로 이관
      - 사용중인 서버가 다운되어도 작업이 인계되어 안정성 상승

### 5.7.3 정리

- 장점
  - 관리자가 한 명이므로 구현 쉬움
  - 슬레이브간 처리 동기화 필요없으므로 통신량이 줄어듬
- 단점
  - 마스터가 없어지면 작업 중단 (작업 인계 필요)
  - 마스터의 부하가 높아진다



## 5.8 압축

### 5.8.1 압축이란? & 5.8.2 자세히 보자

- 쓸데없는 공간을 제거해서 크기를 줄이는 것
- 디지털 데이터의 쓸데없는 공간
  - 중복 패턴을 일정 패턴으로 변경
  - 장점
    - 데이터 크기를 줄임
  - 단점
    - 처리 시간이 걸림 (zip압축의 경우 데이터 전체를 읽어야 함)
- 압축한 데이터를 실제 사용할 때는 원래 상태로 돌려야 함





- 가역 압축
  - 원래 상태로 복구할 수 있는 압축
- 비가역 압축
  - 원래 상태로 복구할 수 없는 손실 압축
  - MP3, JPEG가 대표적
    - 사람이 식별할 수 없는 데이터는 손실시킴

### 5.8.3 어디에 사용되나?

- 자바의 jar
  - 실제로는 zip형식 압축
  - 소스코드를 읽어 애플리케이션 실행코드를 만든 후에는 사용하지 않으므로 첫 실행시에만 시간 소요
- Office파일들
- 이미지 등등...
- 데이터베이스
  - 속도가 중요하므로 간단한 압축구조를 사용
  - 중복제거(deduplication)
    - 같은 파일은 같은 블록을 사용하도록 하여 중복된 데이터는 하나의 디스크 공간만 차지

### 5.8.4 정리

- 불필요한 공간을 제거해서 데이터 크기를 줄이는 기술
- 데이터 처리/교환 시 오버헤드를 줄임
- 압축 해제/처리로 인해 리소스 부하가 높아지거나 처리시간이 증가할 수 있음



## 5.9 오류 체크/오류 수정

### 5.9.1 오류 체크 / 오류 수정이란?

데이터의 망가짐을 방지하기 위한 기술

### 5.9.2 자세히 보자

- 데이터 오류가 발생하는 원인
  - 통신 중에 데이터 파손
    - 전기 신호를 사용한 데이터 교환 시, 잡음으로 데이터가 파손 될 수 있음
  - 칩에서의 데이터 파손
    - 메모리의 데이터도 전기적으로 저장되며, 전기적 영향으로 값이 바뀔 수 있음

### 5.9.3 실수나 오류를 검출하려면

- 오류 검출
  - 정답을 알 필요는 없음
  - 틀렸다는 것만 체크하면 됨
  - 패리티 비트(parity bit)라는 추가 비트를 부여
    - 전체 이진 데이터가 반드시 1이 짝수 또는 홀수가 되도록 마지막 1비트를 추가
      - 1비트의 변경을 감지 할 수 있음
  - 오류 여부를 알기 위해 데이터 양이 증가하며, 오류 체크를 위한 처리에 오버헤드 발생

### 5.9.4 어디에 사용되나?

- CPU, 메모리 내부
- 메모리는 특히 에너지 입자의 영향을 받기 쉬움
  - ECC 메모리
    - 쓰기 처리 시 패리티 비트 계산을 해서 오류 검출 및 수정 기능을 제공
    - 비쌈! 서버용
- 네트워크 통신
  - 프로토콜에 따라 각 계층의 오류 검출 구조를 갖추고 있음
  - TCP/IP, Ethernet
    - 계층별로 체크섬이나 CRC오류를 검출하는 구조를 도입
    - 수신측에서 오류 패킷이 있으면 해당 데이터를 제외하고 재전송하도록 요구(TCP의 역할)

### 5.9.5 정리

- 오류 체크
  - 디지털 데이터의 오류를 스스로 확인하고, 파손되었을 때 이를 감지하는것을 의미
- 오류 수정
  - 파손된 데이터를 자동으로 복구
  - 여러가지 기법이 있음~
    - 리소스 부하 상승
    - 알고리즘의 복잡화