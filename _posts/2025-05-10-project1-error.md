---
title: "1차 프로젝트 정리 - Batch error 처리 테스트"
date: 2025-05-10
description: "1차 프로젝트 Batch 중 error 발생 시 데이터 정합성 테스트해보기"
categories:
  - Workshop
  - 1차 Project
tags:
  - Workshop
  - Project
  - Spring Batch
---

# 배치 오류 처리

- `JpaCursorItemReader`는 상태 관리 기능을 제공하는 `ItemStreamReader`를 상속받아 배치 작업이 오류로 인해 중지되었을 경우, 오류가 났던 부분에서 **재실행**이 가능

### 정리:

- `JpaCursorItemReader`는 `AbstractItemCountingItemStreamItemReader`를 확장하여 **상태 관리 기능**을 제공
- `ItemStreamReader`를 상속받기 때문에  `ExecutionContext`를 사용하여 **현재 읽고 있는 위치**를 추적하고 저장
- 배치 작업이 중간에 **오류**가 발생하여 중단되면, 다음 실행 시 **이전에 읽은 위치부터 재실행**할 수 있음    

<br><br><br>

1. DB 데이터  
![image.png](/assets/post_img/250510/image.png)

2. 의도적으로 에러를 발생시킴
![image.png](/assets/post_img/250510/image1.png)

3. job 파라미터 동일하게 실행
![image.png](/assets/post_img/250510/image2.png)

4. p4 데이터 에러 발생
![image.png](/assets/post_img/250510/image3.png)

5. 배치 결과 테이블에 p4 없음
![image.png](/assets/post_img/250510/image4.png)

6. 의도적으로 만든 예외를 제거하고 재실행 시
![image.png](/assets/post_img/250510/image5.png)

7. 오류난 곳부터 재시작!!!! → 중복없음
![image.png](/assets/post_img/250510/image6.png)

8. BATCH_JOB_EXECUTION 테이블에서 첫번째 작업은 실패, 두번째 작업은 완료된 상태임을 알 수 있음
![image.png](/assets/post_img/250510/image8.png)

9. 같은Job parameter로 실행(첫번째 에러, 두번째 완료) → Job instance는 하나, Job excution은 두개가 생성됨~~
![image.png](/assets/post_img/250510/image7.png)


추가적으로 배치 오류 처리 어떤식으로 할건지 고민
