---
title: "1차 프로젝트 정리 - Trouble shooting"
date: 2025-05-10
description: "1차 프로젝트 트러블 슈팅(배치)"
categories:
  - Workshop
  - 1차 Project
tags:
  - Workshop
  - Project
  - Spring Batch
---
# DB 크론 동적 설정
![image.png](/assets/post_img/250510/image10.png)

- **원인**
  - 웹 서비스 서버 내부에 배치 서버를 포함해서 개발함 (배치할 작업이 많지않아서)
  - Cron 표현식이 하드코딩됨
  - 로컬에서 개발 후 실서버로 옮겼는데 실서버에서 테스트를 위해 Cron 표현식을 수정하는 일이 잦아짐
  - main 브랜치에 PR 후 merge를 하게되면 GitHub Actions를 통해 CI/CD가 자동으로 이루어짐
  - 서버가 재실행 되는 동안 FE, BE 작업이 일시 중단됨

- **해결**

  ![image.png](/assets/post_img/250510/image9.png)

  - DB에서 일정 시간마다 값을 불러와 cron 표현식 변경을 감지하도록 수정
  - 시간을 수정하더라도 작업 중단 안됨

- **추가 개선할 점**
  - **Admin UI 추가**
    - 관리자 페이지를 통해 Cron 표현식 직접 수정 가능
    - 현재 설정된 스케줄 확인 및 수정 내역 관리
  - **이벤트 기반 아키텍처(EDA) 도입**
    - Admin UI에서 cron 수정 시 `스케줄 변경 이벤트` 발행
    - 서버는 해당 이벤트를 수신하고 기존 스케줄을 중단 후 새로운 스케줄로 갱신
    - Polling 없이 실시간 반영 가능
  
---

# 데이터 조회 속도 저하 문제

<div style="display: flex; gap: 10px;">
  <img src="/assets/post_img/250510/image12.png">
  <img src="/assets/post_img/250510/image11.png">
</div>

Paging VS Cursor Reader의 성능 차이를 비교

- **데이터 개수당 속도 비교**(Chunk size 500)
  - 1만개 & 2만개 : Paging > Cursor
  - Paging 느린 이유

    Paging 방식은 `offset`을 이용해 데이터를 `page size`만큼 불러옴

    하지만 `offset`은 원하는 위치부터 바로 읽는 것이 아니라, **앞의 데이터를 모두 스캔 후 버리고** 그 다음 데이터를 반환

    데이터가 많아질수록, offset값이 커질수록 **버려지는 데이터가 많아져** 성능 저하 발생

  - Cursor가 빠른 이유

    Cursor 방식은 `WHERE` 조건을 사용하며 fetch size만큼 데이터를  애플리케이션 메모리에 받아 처리

    OOM(Out of Memory)가 발생할 가능성이 있지만 불필요한 스캔이 없어 Paging 보단 빠른 속도


- Chunk size 당 속도 비교
  - **전제 조건** : Paging의 page size = Cursor의 chunk size
  - **50~500** : Cursor 기반이 Paging 기반보다 월등히 속도가 빠른 것을 알 수 있음
  - **1000~5000** : 속도 차이가 별로 없음
  - **속도 차이의 이유**
    - 작은 chunk에서는 offset을 자주 호출해서 느림
    - **1000~5000**
      Paging: 여전히 offset 스캔은 존재하나, **offset 횟수 자체 감소**
