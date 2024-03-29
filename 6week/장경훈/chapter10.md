# 알림 시스템 설계

## 알림 유형별 지원 방안
### IOS 푸시 알림 
![](https://velog.velcdn.com/images/haron/post/b59cafde-f1da-46a1-830d-7ac322050803/image.png)

- 알림 제공자 : 알림 요청으로 푸시 알림 서비스로 보내는 주체(단말 토큰, 페이로드 필요)
- APNS: 애플이 제공하는 원격 서비스 , 푸시 알림을 IOS에 보냄
- IOS 단말: 푸시 알림을 수신하는 단말기

### 안드로이드 푸시 알림
![](https://velog.velcdn.com/images/haron/post/3592b9e6-a6e2-4f0b-a31a-d3c5c305adca/image.png)
- 애플과 비슷한 형태인것을 확인 가능, <strong>SMS 메세지, 이메일도 같은 구성</strong>

## 연락처 정보 수집 절차
![](https://velog.velcdn.com/images/haron/post/d4ea5ac3-307e-4165-bbeb-a7ab7c15e177/image.png)

## 개략적 설계안 초안 
![](https://velog.velcdn.com/images/haron/post/c3d6a47f-99e3-43ae-b688-c4ee728524d9/image.png)
### 위의 설계의 문제점
- SPOF : 알림 서비스에 서버가 하나밖에 없어 장애조치가 어려움
- 규모 확장성: 한 대 서비스로 푸시 알림에 관계된 모든 것을 처리하기 때문에 규모를 개별적으로 늘리기 어려움
- 성능 병목: 트래픽이 많이 몰리면 과부화 상태에 빠질 수 있다.

## 개선 버전
![](https://velog.velcdn.com/images/haron/post/d26842ef-18af-4793-a87d-390fa38b0efd/image.png)
- 데이터베이스와 캐시를 주 서버에서 분리
- 알림 서버 증설 및 오토스케일링 적용
- 메시지 큐를 사용해 컴포넌트 사이 강한 결합 제거
### 알림 서버의 역할
- 알림 전송 API: 스팸 방지를 위해 보통 사내 서비스 또는 인증된 클라이언트만 이용 가능하다.
- 알림 검증: 이메일 주소, 전화번호 등에 기본적 검증을 수행한다.
데이터베이스 또는 캐시 질의: 알림에 포함시킬 데이터를 가져오는 기능이다.
- 알림 전송: 알림 데이터를 메세지 큐에 넣는다. 본 설계안의 경우 하나 이상의 메세지 큐를 사용하므로 알림을 병렬적으로 처리할 수 있다.
- 캐시: 사용자 정보, 단말 정보, 알림 템플릿 등을 캐시한다.
- 데이터베이스: 사용자, 알림, 설정 등 다양한 정보를 저장한다.
- 메세지 큐: 시스템 컴포넌트 간 의존성을 제거하기 위해 사용한다. 다량의 알림이 전송되어야 하는 경우를 대비한 버퍼 역할도 한다.
- 작업 서버: 메세지 큐에서 전송할 알림을 꺼내서 제 3자 서비스로 전달하는 역할을 담당하는 서버다.

### 알림이 전송되는 과정
1. API를 호출하여 알림 서버로 알림을 보낸다.
2. 알림 서버는 사용자 정보, 단말 토큰, 알림 설정 같은 메타데이터를 캐시나 데이터베이스에서 가져온다.
3. 알림 서버는 전송할 알림에 맞는 이벤트를 만들어서 해당 이벤트를 위한 큐에 넣는다.
4.작업 서버는 메세지 큐에서 알림 이벤트를 꺼낸다.
5. 작업 서버는 알림을 제3자 서비스로 보낸다.
6. 제 3자 서비스는 사용자 단말로 알림을 전송한다.

## 상세 설계
### 안정성(고려해야 할 점)
- 데이터 손실 방지
- 알림 중복 전송 방지

### 추가로 필요한 컴포넌트 및 고려 사항 
- 알림 템플릿
- 알림 설정
- 전송률 제한
- 재시도 방법
- 푸시 알림과 보안
- 큐 모니터링
- 이벤트 추적

## 수정된 설계안 
![](https://velog.velcdn.com/images/haron/post/83ceae6c-83d6-45a2-9ac5-4225a738d3e2/image.png)
- 알림 서버에 인증(authentication)과 전송률 제한(rete-limiting 기능이 추가되었다.
- 전송 실패에 대응하기 위한 재시도 기능이 추가되었다. 전송에 실패한 알림은 다시 큐에 넣고 지정된 횟수만큼 재시도한다.
- 전송 템플릿을 사용하여 알림 생성 과정을 단순화하고 알림 내용의 일관성을 유지한다.
- 모니터링과 추적 시스템을 추가하여 시스템 상태를 확인하고 추후 시스템을 개선하기 쉽도록 하였다.


## 마무리
- 안정성: 메세지 전송 실패율을 낮추기 위해 안정적인 재시도 메커니즘을 도입하였다.
- 보안: 인증된 클라이언트만이 알림을 보낼 수 있도록 appKey, appSecret 등의 메커니즘을 이용하였다.
- 이벤트 추적 및 모니터링: 알림이 만들어진 후 성공적으로 전송되기까지의 과정을 추적하고 시스템 상태를 모니터링하기 위해 알림 전송의 각 단계마다 이벤트를 추적하고 모니터링할 수 있는 시스템을 통합하였다.
- 사용자 설정: 사용자가 알림 수신 설정을 조정할 수 있도록 하였다. 따라서 알림을 보내기 전 반드시 해당 설정을 확인하도록 시스템 설계를 변경하였다.
- 전송률 제한: 사용자에게 알림을 보내는 빈도를 제한할 수 있도록 하였다.
