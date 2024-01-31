# 개략적인 규모 추정

개략적인 규모 추정 (back-of-the-envelope estimation)

- 성능 수치상에서 사고 실험을 행하여 추정치를 계산하는 행위
- 어떤 설계가 요구사항에 부합할 것인지 보기 위하여

`❗️규모 확장성에 대한 표현 → 2의 제곱수, 응답지연 값, 가용성에 관계된 수치들`

## 1️⃣ 2의 제곱수
![CamScanner 2024-01-31 12 44(1)_8](https://github.com/NoMyeongEun/24-1_Study_Large-ScaleSystemDesign/assets/90135669/87cc0283-7929-42db-938b-2f5182f8800c)

## 2️⃣ 모든 프로그래머가 알아야 하는 응답지연 값
![CamScanner 2024-01-31 12 44(1)_9](https://github.com/NoMyeongEun/24-1_Study_Large-ScaleSystemDesign/assets/90135669/a4ce56a6-0631-455f-b4f8-333890e12f5b)
![CamScanner 2024-01-31 12 44(1)_10](https://github.com/NoMyeongEun/24-1_Study_Large-ScaleSystemDesign/assets/90135669/a42d9fc7-1eb1-412a-acf5-02a12e4f2532)

## 3️⃣ 가용성에 관계된 수치들
![CamScanner 2024-01-31 12 44(1)_11](https://github.com/NoMyeongEun/24-1_Study_Large-ScaleSystemDesign/assets/90135669/1da2a8e1-8c6d-438e-9744-da34f8d85d1b)
- 고가용성 : 시스템이 오랜 시간 동안 지속적으로 중단 없이 운영될 수 있는 능력
- SLA : 서비스 사업자와 고객 사이에 맺어진 합의

## 4️⃣ 팁

- 근사치를 활용한 계산
- 가정 적어두기
- 단위 붙이기
- QPS, 최대 QPS, 저장소 요구량, 캐시 요구량, 서버 수 추정
