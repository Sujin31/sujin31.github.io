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

# 실습

개발환경

- Spring Boot 3.4.4
- Spring Batch 5.2.2
- Java 17
- Gradle
- Mysql 8.3

---

## 1. 의존성 주입(build.gradle)

```gradle
  plugins {
      id 'java'
      id 'org.springframework.boot' version '3.4.4'
      id 'io.spring.dependency-management' version '1.1.7'
  }

  group = 'org.example'
  version = '0.0.1-SNAPSHOT'

  java {
      toolchain {
          languageVersion = JavaLanguageVersion.of(17)
      }
  }
  
  configurations {
      compileOnly {
          extendsFrom annotationProcessor
      }
  }
  
  repositories {
      mavenCentral()
  }
  
  dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-batch'
      implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
      compileOnly 'org.projectlombok:lombok'
      runtimeOnly 'com.h2database:h2'
      runtimeOnly 'com.mysql:mysql-connector-j'
      annotationProcessor 'org.projectlombok:lombok'
      testImplementation 'org.springframework.boot:spring-boot-starter-test'
      testImplementation 'org.springframework.batch:spring-batch-test'
      testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
  }
  
  tasks.named('test') {
      useJUnitPlatform()
  }

```

---

## 2. 일일집계 Job 생성

### 2-1. 패키지 정리

![image.png](/assets/post_img/250427/image3.png)

- 클린 아키텍처( 또는 계층형 아키텍처)를 토대로 역할을 나눔
- **application :** 배치의 흐름과 유즈케이스 정의
  - 배치 Job, Step, Tasklet, Reader, Processor, Writer 정의
  - 실제로 "하루치 데이터 집계"라는 유즈케이스를 어떻게 실행할지 구성
  - 비즈니스 로직을 **직접 처리하지 않고**, 필요한 로직은 `domain` 계층에 위임
- **domain :** 핵심 비즈니스 로직
  - “집계”의 **비즈니스 개념**을 담당
  - 재사용 가능하고, 테스트하기 쉬운 구조
- **infrastructure :** 기술적인 세부 구현
  - DB 접근, 외부 API 통신 등
  - Spring Data JPA Repository, JdbcReader/Writer 같은 구현체 포함

    ---


## 3. Job 생성

### Job 생성방식

1. JobBuilder
  - 일반적 방식, `Step`들을 순차적으로 연결하거나 조건에 맞게 흐름 정의
2. FlowBuilder
  - 복잡한 흐름이나 조건 분기를 구현할 떄 사용, 여러 `Step`을 병렬 또는 조건에 따라 실행하도록 설정
3. JobLauncher와 Programmatic Job 실행
  - 프로그램에서 `JobLauncher`를 사용하여 `Job`을 동적으로 실행할 수 있음, 실행 시 동적으로 파라미터를 넘기거나 실행 상태를 제어할 때 유용

### JobBuilder 를 사용해 Job 생성

```java
@Bean
public Job dailyAggregationJob(JobRepository jobRepository, PlatformTransactionManager transactionManager){
    return new JobBuilder("dailyAggregationJob", jobRepository)
            .start(dailyAggregationStep(jobRepository,transactionManager))
//               .next(nextStep(jobRepository,transactionManager))
            .build();
}

```

- **`JobBuilder**("DailyAggregationJob", jobRepository)`
  - `DailyAggregationJob` 이란 이름의 Job 생성
- **`JobRepository`**
  - Spring Batch의 **중앙 데이터 저장소**로서 **배치 작업의 실행 상태**를 추적하고 저장하는 역할
  - 주로 각 `Job` 실행의 상태, 결과 등을 기록
  - 배치 작업의 실행 상태, 예외, 트랜잭션 상태 등을 관리하는데 필수
  - **배치 작업의 실행 로그**를 DB에 저장하거나, 실행 중인 `Step` 의 상태를 유지하는 데 사용
- **`PlatformTransactionManager` 인터페이스**
  - Spring의 **트랜잭션 관리** 담당
  - Spring Batch에서 `Step` 을 실행하는 동안 트랜잭션 처리를 관리
  - **트랜잭션 관리**는 Spring Batch에서 **원자성**을 보장하며, 배치 작업이 중간에 실패할 경우 트랜잭션을 롤백하여 데이터를 일관되게 유지
  - 구현체로는
    - `DataSourceTransactionManager` : **JDBC** 기반의 트랜잭션 관리자로, **DataSource**를 기반으로 트랜잭션을 관리
    - `JpaTransactionManager` : **PA** 기반의 트랜잭션 관리자로, JPA(Hibernate 등)에서 발생하는 트랜잭션을 처리
    - `HibernateTransactionManager`, `KafkaTransactionManager` 등등…
- **`start()`**, **`next()`**
  - `JobBuilder`의 `start()` 메서드는 **Job의 첫 번째 Step**을 정의
  - `next()` 메서드는 **이전 `Step`이 성공적으로 완료된 후** 다음에 실행할 `Step`을 설정
  - `next()`는 여러 `Step`을 순차적으로 연결하여 실행 흐름을 설정하는 데 사용

      ```java
      jobBuilder.start(step1)  // 첫 번째 Step
                 .next(step2)  // 두 번째 Step
                 .next(step3)  // 세 번째 Step
                 .build();
      ```


## 4. Step 생성

### Step 생성 방식

- Tasklet 방식
- Chunk 지향 처리

### 4-1. Tasklet

- **Tasklet 방식 특징**
  - **단일 작업**을 처리하는 방식으로, **상태 유지가 필요 없는 작업**에 적합(ex. 파일 삭제, 로그 기록, 데이터베이스 정리 등)
  - 작업이 한 번에 완료되며, 반복 작업 X
  - 예외처리나 트랜잭션 관리 등의 로직을 명시적으로 작성 할 수 있음

```java
 @Bean
public Step dailyAggregationStep(JobRepository jobRepository, PlatformTransactionManager transactionManager){
	return new StepBuilder("dailyAggregationStep", jobRepository)
	        .tasklet((contribution, chunkContext) -> {
	            log.info("this is tasklet");
	            return RepeatStatus.FINISHED;
	        }, transactionManager)
	        .build();
}
```

- `RepeatStatus.FINISHED` : Tasklet이 정상적으로 종료되었음을 나타냄, 작업이 반복되어야한다면 `RepeatStatus.CONTINUABLE`을 반환

### 4-2. Chunk 지향 처리 방식

- **Chunk 지향 처리 방식 특징**
  - 데이터를 **일정 크기**의 **청크(chunk)**로 나누어 처리하며, 각 청크마다 **읽기(Read) → 처리(Process) → 쓰기(Write)** 작업을 수행하고, 각 청크가 완료될 때마다 **커밋(commit)**
  - **대량 데이터 처리**에 적합
  - **트랜잭션 관리**가 각 청크에 대해 이루어지므로, 데이터 처리 중에 오류가 발생해도 문제를 최소화

```java

private final int CHUNK_SIZE = 500;

@Bean
public Step dailyAggregationStep(JobRepository jobRepository, PlatformTransactionManager transactionManager){
  return new StepBuilder("dailyAggregationStep", jobRepository)
        .<DailyAggregation, DailyAggregation>chunk(CHUNK_SIZE, transactionManager)
        .reader(dailyAggregationReader())
        .processor(dailyAggregationProcessor())
        .writer(dailyAggregationWriter())
        .build();
}

@Bean
public ItemReader<DailyAggregation> dailyAggregationReader() {
  return new MyItemReader();  // 데이터를 읽어오는 로직
}

@Bean
public ItemProcessor<DailyAggregation, DailyAggregation> dailyAggregationProcessor() {
  return new MyItemProcessor();  // 데이터를 처리하는 로직
}

@Bean
public ItemWriter<DailyAggregation> dailyAggregationWriter() {
  return new MyItemWriter();  // 데이터를 저장하는 로직
}
```

- `chunk(CHUNK_SIZE, transactionManager)` : 한번에 처리할 청크 크기 설정, 예시에는 500개
- `ItemReader` : 데이터를 읽어오는 역할, 한줄씩 읽어옴
- `ItemProcessor` : 읽어온 데이터를 처리하는 역할, 한줄씩 처리됨
- `ItemWriter` : 처리된 데이터를 저장하는 역할, 청크만큼 데이터가 쌓이면 실행
- 각 청크가 완료될 때마다 커밋

---

## 5. 실행

### 5-1. 실행 전! 메타 테이블 생성

기본적으로 H2 DB를 사용할 경우, 스프링 배치 메타 테이블을 Boot가 실행될때 자동으로 생성해주지만, **MySQL이나 Oracle과 같은 DB를 사용할때는 개발자가 직접 생성**해야함!

- schema를 검색해 DBMS에 맞춰 메타테이블 생성해줌

![image.png](/assets/post_img/250427/image2.png)

mysql schema

![image.png](/assets/post_img/250427/image1.png)

생성된 batch meta table

### 5-2. 진짜 실행

![image.png](/assets/post_img/250427/image.png)
