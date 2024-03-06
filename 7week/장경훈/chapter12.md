# 채팅 시스템 설계
## 요구사항
- 응답지연이 낮은 일대일 채팅 기능
- 최대 100명까지 가능한 그룹 채팅 기능
- 사용자 접속상태 표시 기능
- 다양한 단말 지원 및 한 계정 여러 - 단말 동시 접속 지원
- 푸시 알림
- 5천만 DAU 지원

## 개략적 설계안 제시 및 동의 구하기
![](https://velog.velcdn.com/images/ohdowon064/post/adc28b6d-eeb2-4a95-92e0-2fbbe0988b2a/image.png) 
- 클라이언트로부터 메세지 수신
- 메시지 수신자 결정 및 전달
- 수신자가 접속 상태가 아닌 경우, 메세지 보관

### 폴링
![](https://velog.velcdn.com/images/ohdowon064/post/a0f620b4-000f-45ea-a102-176c15d55241/image.png)
- 클라이언트가 주기적으로 서버에게 메시지 여부를 물어보는 방식이다.
- 폴링 비용은 주기가 짧을 수록 올라간다.
- 답할 메시지가 없을 경우 서버 자원이 낭비된다.

### 롱 폴링
![](https://velog.velcdn.com/images/ohdowon064/post/0c47849e-5aeb-42ea-89cf-aea442d04340/image.png)
- 폴링 단점 보완 기법
- 메세지 반환 or 타임아웃 까지 연결 유지
- 메세지를 받으면 기존 연결 종료 및 서버에 새로운 요청을 보낸다.<Br><br>

<strong>약점</strong>
- 송신자와 수신자가 같은 채팅 서버에 접속하지 않을 수 있다.
- HTTP 서버들은 대부분 stateless 서버이며, 로드 밸러싱을 위해 라운드 로빈 알고리즘을 사용하는 경우, 송신자와 수신자가 다른 채팅 서버를 사용할 수 있다.
- 서버 입장에서 연결 해제 여부를 알 수 없다.
- 메시지를 받지않은 클라이언트도 타임아웃이 일어날 때마다 주기적으로 서버에 재접속해야하므로 여전히 비효율적이다.

## 웹 소켓
![](https://velog.velcdn.com/images/ohdowon064/post/bd27d8c8-6b33-48be-8b54-6abca899d665/image.png)
- 웹 소켓은 서버가 클라이언트에게 비동기 메시지를 보낼 때 가장 널리 사용하는 기술이다.
- 연결은 클라이언트가 시작한다.
- 처음에 HTTP 연결 후, 특정 handshake 절차를 거치고 웹 소켓 연결로 업그레이드된다.
- 웹 소켓 연결은 항구적이며 양방향이다.
- HTTP/HTTPS의 80 또는 443 포트를 그대로 사용하기 때문에 방화벽 환경에서도 동작한다.

## 개략적 설계안
![](https://velog.velcdn.com/images/ohdowon064/post/b58345e2-dcf3-42d1-b6d3-f63cc12a0330/image.png)

### stateless 서비스
- 로그인, 회원가입, 사용자 프로필 등 전통적인 요청/응답이 해당된다.
- 무상태 서비스는 로드밸런서 뒤에 위치한다.
- 서비스 탐색(service discovery) 서비스는 클라이언트가 접속할 채팅 서버의 DNS 호스트명을 알려주는 서비스이다.

### stateful 서비스
- 채팅 서비스가 해당된다.
- 각 클라이언트와 채팅 서버는 독립적인 네트워크 연결을 유지해야한다.
클라이언트는 서버와 연결이 살아있는 한 다른 서버로 연결을 변경하지 않는다.
- 서비스 탐색 서비스는 채팅 서비스와 협력하여 특정 채팅 서버에 부하가 몰리지 않도록 한다.

### third-party 연동
- 푸시 알림이 해당된다.
- 앱이 실행 중이 아니어도 알림을 받아야한다.

## 규모 확장성
![](https://velog.velcdn.com/images/ohdowon064/post/9606f457-33bf-4fb4-9caf-c23551857bb2/image.png)

- 채팅서버는 클라이언트 간 메시지 중계를 담당한다.
- 접속상태(presence) 서버는 사용자 접속 여부를 관리한다.
- API 서버는 채팅을 제외한 로그인, - 회원가입 프로필 변경 등 전부를 처리한다.
- 알림 서버는 푸시 알림을 보낸다.
키-값 저장소에는 채팅 이력을 보관한다.

## 저장소 선택
- <strong>어떤 데이터베이스를 사용할 것인지는 데이터의 유형과 읽기/쓰기 연산의 패턴을 봐야한다.</strong>
- 일반 데이터는 안정성 보장을 위해 관계형 데이터베이스에 보관
- 채팅 이력 읽기/쓰기 연산을 보면 처리해야하는 메세지가 많기 때문에 키-값 저장소를 추천

## 상세 설계
### 서비스 탐색
![](https://velog.velcdn.com/images/ohdowon064/post/85e3d0b0-8221-48b5-a6ca-d37f0d8f2728/image.png)
- 클라이언트에게 가장 적합한 채팅 서버를 추천하는 것

### 메시지 흐름 
![](https://velog.velcdn.com/images/ohdowon064/post/b2ffd01b-5ac8-4131-b806-02729774546c/image.png)

### 여러 단말 사이의 메세지 동기화
![](https://velog.velcdn.com/images/ohdowon064/post/2f48cd75-399f-4107-b54d-281be22e9e4f/image.png)

### 소규모 그룹 채팅 메세지 흐름 
![](https://velog.velcdn.com/images/ohdowon064/post/fe46cfee-7417-467c-9801-8c9bf7cf6308/image.png)
![](https://velog.velcdn.com/images/ohdowon064/post/5b357035-b85d-4593-8487-b55a56516f8c/image.png)
### 접속상태 표시

#### 로그인
![](https://velog.velcdn.com/images/ohdowon064/post/5701fa72-84ad-4b28-a70a-3bd7f328fc1b/image.png)
#### 로그아웃
![](https://velog.velcdn.com/images/ohdowon064/post/e5583efc-07f0-4ceb-b087-d06485c07c87/image.png)
#### 접속 장애
![](https://velog.velcdn.com/images/ohdowon064/post/9e839fbe-ea36-43a8-aafd-caac31b1edc4/image.png)
#### 상태정보의 전송
![](https://velog.velcdn.com/images/ohdowon064/post/a992e4fa-822a-488d-96b0-55c4e12874d5/image.png)

## 마무리
### 추가 논의 사항
- 미디어 파일 지원 방법
- 종단 간 암호화
- 캐시
- 로딩 속도 개선
- 오류 처리
