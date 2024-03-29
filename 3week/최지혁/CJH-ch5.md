# 해시 키 재배치 문제

이번 장에서는 수평적 규모 확장성을 달성하기 위해 요청이나 데이터를 서버에 균등하게 나누는 법에 관해서 설명하고 있다.

# 안정해시
해시테이블 크기가 조정될때 평균적으로 오직 k/n개의 키만 재배치 하는 해시 기술이다.
- k: 키의 개수
- n: 슬롯의 개수

#### 해시 공간과 해시 링
해시 함수 f로는 SHA-1을 사용한다고 가정하자. SHA-1의 해시 공간 범위는 0부터 2^160-1까지라고 알려져있다.즉 x0은 0, xn은 2^160-1이다.

해시 공간을 구부려 접어 해시 링을 만들 수 있다.
![](https://jonghoonpark.com/assets//images/2023-05-25-안정-해시-설계/image2.png)
![](https://jonghoonpark.com/assets//images/2023-05-25-안정-해시-설계/image3.png)
- 위 해시 링에 서버와 키를 배치할 수 있으며 기본적으로 아래 그림처럼 시계 방향으로 링을 탐색해 가며 서버에 키를 저장한다.
![](https://jonghoonpark.com/assets//images/2023-05-25-안정-해시-설계/image6.png)

위 그림에서는 서버의 추가와 제거가 자연스러운데 추가 혹은 제거시 단 순히 키만 재배치가 되기에 나머지 키에는 영향을 주지 않는다.

#### 기본 구현법의 두가지 문제
기본 구현 법에는 두가지 분제가 있다.
- 서버의 추가 및 삭제시 파티션을 균등하게 유지하는게 불가능하다.
- 당연하지만 키의 균등 분포도 달성하기 어렵다.

![](https://jonghoonpark.com/assets//images/2023-05-25-안정-해시-설계/image9.png)

#### 가상노드
위 문제를 해결하기 위해 나온 것이 바로 가상 노드이다.
가상 노드랑 실제 서버를 가리키는 노드로서 하나의 서버가 여러개의 가상 노드를 가질 수 있다. 


![](https://jonghoonpark.com/assets//images/2023-05-25-안정-해시-설계/image10.png)
위 그림 처럼 시계방향 탐색 시 가장 먼저 만나는 가상 노드가 키가 저장될 서버에 해당 한다.

- 가상 노드의 개수를 늘리면 키의 분포는 점점 더 균등해진다. 표준편차가 작아짐.
- 다만 가상 노드 다체의 데이터를 저장할 공간은 많이 필요하기 때문에 tradeoff를 고려해야 함.

#### 안정해시의 이점
- 서버 수의 변경이 있을 때 재배치되는 키의 수 최소화
- 데이터의 균둥 분포를 도모해 수평적 슈모 확장성 달성이 용이함.
- 핫스팟 문제를 줄인다.
	- 유명인사 문제가 발생시 데이터가 균등하게 분포되므로 대기시간을 최소화 함.