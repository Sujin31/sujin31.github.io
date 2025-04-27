---
title: "Spring Batch 5.x - 1"
date: 2025-04-20
description: "Spring Batch 5버전 사용하면서 기본 지식, 실습, 프로젝트의 정리"
categories:
  - Spring
  - Spring Batch
tags:
  - Spring Batch
  - Project
---

# **Spring Batch?**

## 배치 애플리케이션이란

> 배치(Batch)는 **일괄처리** 란 뜻
>

만약,

매일 전날의 대규모 데이터를 집계 해야한다면?

→ 일반적인 **웹 애플리케이션 (Tomcat + Spring MVC)** 에서 처리한다면

→ CPU, I/O 등의 자원을 많이 사용하게 되어 **다른 요청 처리에 영향**
→ 집계는 **하루에 한 번** 수행되는데, 이를 위해 API를 구성하는 건 **비효율적**

데이터가 너무 많아서 처리 중 실패가 난다면?

→ 5만번째에서 실패했다면, **5만 1번째에서 다시 실행할 수는 없을까**

실수로 2번 실행되는 문제가 있다면?

→**같은 파라미터로 실행 시 이미 실행된 적이 있으면 중복 방지 필요**

---
### 🔧 웹 애플리케이션처럼 배치도 프레임워크가 있으면?  
  
# **➡️ Spring Batch!!**  <br><br><br>
  
> **배치 어플리케이션의 조건**  

- 대용량 데이터 - 배치 어플리케이션은 대량의 데이터를 가져오거나, 전달하거나, 계산하는 등의 처리를 할 수 있어야 합니다.
- 자동화 - 배치 어플리케이션은 심각한 문제 해결을 제외하고는 **사용자 개입 없이 실행**되어야 합니다.
- 견고성 - 배치 어플리케이션은 잘못된 데이터를 충돌/중단 없이 처리할 수 있어야 합니다.
- 신뢰성 - 배치 어플리케이션은 무엇이 잘못되었는지를 추적할 수 있어야 합니다. (로깅, 알림)
- 성능 - 배치 어플리케이션은 **지정한 시간 안에 처리를 완료**하거나 동시에 실행되는 **다른 어플리케이션을 방해하지 않도록 수행**되어야합니다.

---

## Batch vs Quartz

- **Quartz는 스케줄러의 역할**이지, Batch 와 같이 **대용량 데이터 배치 처리에 대한 기능을 지원** ❌


    | 항목 | **Spring Batch** | **Quartz Scheduler** |
    | --- | --- | --- |
    | **주요 목적** | **대량의 데이터 처리, ETL, 일괄처리** | **정기적인 작업 스케줄링 (알람, 잡 예약)** |
    | **작업 유형** | 데이터 읽기 → 처리 → 저장 (예: CSV → DB) | 알람 발송, DB 정리, 이메일 전송 등 |
    | **반복 실행** | 직접 스케줄링 도구 필요 (`Scheduler`, `JobLauncher`) | 내부적으로 스케줄링 처리함 (cron, 반복 등) |
    | **중단/재시작 기능** | ✅ 있음 (ExecutionContext로 상태 저장) | ⛔ 없음 (실패 시 별도 처리 필요) |
    | **상태 저장** | JobExecution, StepExecution 등으로 상태 추적 가능 | 기본은 무상태(stateless), DB로 상태 저장 설정 가능 |
    | **사용 대상** | 대규모 배치 작업에 특화 | 간단한 정기 작업 예약에 특화 |
    | **Spring 지원** | Spring Batch 모듈 | Quartz + Spring 연동 가능 (`spring-boot-starter-quartz`) |

---

## 📌용어 정리

### Job

- 배치 처리 과정을 하나의 단위로 만들어 놓은 객체
- 배치 처리 과정에 있어 전체 계층 최상단에 위치

### JobInstance

- `Job`의 한 번의 실행 단위
- `Job`을 실행 시키면 하나의 `JobInstance` 생성
- `JobParameters` 로 구분

### JobParameters

- Spring Batch에서 `Job`을 실행할 때 넘겨주는 매개변수
- `JobInstance` 를 유일하게 구분시키기 위한 키 값
- `String`, `Double`, `Long`, `Date` 4가지 형식만을 지원

### JobExecution

- `JobInstance` 의 실행 기록 (시작, 종료, 상태 등)
- `JobInstance` 가 재실행을 해도 동일한 `JobInstance` 를 실행 시키지만 `JobExecution` 는 개별 생성됨

### Step

- `Job`의 하나의 **작업 단위**
- **순차적인 처리 단계**를 캡슐화함
- 최소 1개 이상의 Step이 있어야 Job이 동작 가능
- 각 Step 내부에서 Reader → Processor → Writer로 동작
- 실제 배치 처리의 핵심 로직이 들어있는 곳

### StepExecution

- `Step` 이 실제로 실행될 때 생성되는 객체
- `JobExecution` 이 실행되면, 각 `Step` 마다 별도의 `StepExecution`이 생성
- **실행 로그, read/write/skip/rollback 등 처리 현황**을 담고 있음
- 실패한 Step 이후의 Step은 실행되지 않기 때문에, **StepExecution도 생성되지 않음**
- **실제 실행 시점에만 생성됨**

### ExecutionContext

- `Job` 또는 `Step` 간에 **데이터를 공유할 수 있는 저장소 (Map 형태)**
- **JobExecutionContext**: Job 전체에서 공유됨
  - Commit 시점에 DB에 저장
- **StepExecutionContext**: Step 내에서 공유됨
  - Step 실행 중에 저장
- 배치 실패 후 재실행 시, **마지막 처리 지점을 복원**하는 데 사용

### JobRepository

- `Job`, `JobInstance`, `JobExecution`, `StepExecution`, `ExecutionContext` 등의 **모든 실행 정보를 DB에 저장하고 관리**하는 컴포넌트
- 배치 실행 상태, 이력, 중복 실행 방지 등을 위해 **DB에 메타데이터 저장**
- Spring Batch가 상태를 추적하고 복구할 수 있도록 도와줌

### JobLauncher

- `Job`을 실행하는 역할
- `Job`과 `JobParameters`를 전달받아 실행

### ItemReader<T>

- Step 내에서 **입력 데이터를 읽는 인터페이스**
- 매 호출마다 **하나의 아이템을 반환** (`T read()`)
- 더 이상 읽을 데이터가 없으면 `null` 반환

### ItemProcessor<I, O>

- `Reader`에서 읽은 데이터를 **가공/필터링/검증**하는 컴포넌트
- 반환값이 `null` 이면 해당 아이템은 `Writer` 로 넘어가지 않음
- 선택적 컴포넌트

### ItemWriter<O>

- `Processor` 를 거친 데이터를 **최종적으로 저장**하는 컴포넌트
- 보통 List 형태로 넘어오고, 내부에서 한 번에 처리 (`write(List<? extends O>)`)
- 대상 : DB저장, 파일 기록, 메시지 큐 전송 등
- **Chunk 단위로 모아서 처리**
