# 검색어 자동완성 시스템
## 요구사항
- 빠른 응답 속도 : 시스템 응답속도는 100밀리초 이내여야 한다.
- 연관성 : 자동완성 검색어는 입력한 단어와 연관되어야 한다.
- 정렬 : 계산 결과는 인기도 등의 순위 모델에 의해 정렬되어야 한다.
- 규모 확장성 : 많은 트래픽을 감당할 수 있도록 확장 가능해야 한다.
- 고가용성 : 시스템 일부에 장애, 예상치 못한 문제가 생겨도 계속 사용 가능해야 한다.

## 개략적 설계안 제시 및 동의 구하기
### 데이터 수집 서비스
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/6f0ff5ee-1303-4630-be33-842f3e71cefd)
### 질의 서비스
```SQL
SELECT * FROM frequency_table
WHERE query Like `prefix%`
ORDER BY frequency DESC
LIMIT 5;
```

## 상세 설계
### 트라이 자료구조
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/9fc336d7-36dd-4612-afe1-7d9213f3aee0)
- 트리 형태의 자료구조
- 루트 노드는 빈 문자열
- 각 노드는 글자 하나를 저장 26개의 자식 가질 수 있음
- 트리 노드는 하나의 단어 , 접두어 문자열을 나타냄

### 자동 완성 구연 방법
p: 접두어(prefix)의 길이<br>
n: 트라이 안에 있는 노드 개수<br>
c: 주어진 노드의 자식 노드 개수<Br>
알고리즘 복잡도 = O(p) + O(c) + O(clogc)
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/5d5b4f50-6b2d-4842-bab0-ea5f8466e29f)
<strong>최악의 경우 전체 트라이를 다 검색해야 한다.</strong>

### 접두어 최대 길이 제한
사용자가 검색창에 긴 검색어를 입력하는 일은 거의 없음.

따라서 p값은 작은 정숫값이라고 가정해도 안전하다.

→ 시간복잡도는 O(p)에서 O(작은 상숫값) = O(1)로 바뀔 것이다.

### 노드에 인기 검색어 캐시
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/a4349c00-1359-418f-ab19-16cb12b8db0e)

### 데이터 수집 서비스 
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/e613c44e-ff25-4285-b1c2-c19f3d77288d)

### 트라이 데이터 베이스 
1. 문서 저장소
- 새 트라이를 매주 만들 것이므로 주기적으로 트라이를 직렬화하여 데이터베이스에 저장할 수 있음
- 몽고디비 같은 문서 저장소를 활용 가능
2. 키-값 저장소
- 해시 테이블 형태로 변환 가능
- 트라이에 보관된 모든 접두어를 해시 테이블 키로 변환
- 각 트라이 노드에 보관된 모든 데이터를 해시 테이블 값을 변환

![](https://github.com/sangminlee98/system-design-interview/assets/83197138/5e550841-db93-4e81-abfe-e77a72a94df7)

### 질의 서비스
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/fe954ee6-7139-4f0e-92ad-6337e4afb121)

#### 질의서비스 최적화 방안
- AJAX 요청:
요청을 보내고 받기 위해 페이지를 새로고침 할 필요가 없다.
- 브라우저 캐싱:
제안된 검색어들을 브라우저 캐시에 넣어두면 후속 질의의 결과는 해당 캐시에서 바로 가져갈 수 있다.
데이터 샘플링:
- 모든 질의 결과를 로깅하도록 하면 CPU 자원과 저장공간을 많이 소진하게 된다.
데이터 샘플링 기법은 N개 요청 가운데 1개만 로깅하도록 하는 것.

### 트라이 연산
#### 트라이 생성
- 작업 서버가 담당, 데이터를 이용해 트라이 생성
#### 트라이 갱신
- 매주 한 번 갱신하는 방법
- 트라이의 각 노드를 개별적 갱신하는 방법: 성능이 좋지 않다, 트라이가 작을때 고려해볼만 함
#### 검색어 삭제
![](https://github.com/sangminlee98/system-design-interview/assets/83197138/fd7749a6-4471-4d8e-a876-da38e0853b4b)

### 저장소 규모 확장
- 두 대의 서버가 필요한 경우: ‘a’부터 ‘m’까지로 시작하는 검색어는 첫 번째 서버에, 나머지는 두 번째 서버에 저장한다.
- 세 대의 서버가 필요한 경우: ‘a’부터 ‘i’까지는 첫 번째 서버에, ‘j’부터 ‘r’까지는 두 번째 서버에, 나머지는 세 번째 서버에 저장한다.

