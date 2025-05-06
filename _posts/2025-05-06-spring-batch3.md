---
title: "Spring Batch 5.x - 3"
date: 2025-05-06
description: "Spring Batch 5버전 메타테이블 & 오류 처리"
categories:
  - Spring
  - Spring Batch
tags:
  - Spring Batch
  - Project
---

# 사전 준비

## 테스트 테이블 생성

```sql
@Entity
public class Product {
    @Id
    private Long id;
    private String code;
    private Double price;
}
```

![image.png](/assets/post_img/250506/image.png)

# 실행

## 1. Job 스케줄러 등록

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class BatchScheduler {
    private final JobLauncher jobLauncher;
    private final Job dailyAggregationJob;

    @Scheduled(cron = "0/10 * * * * *")
    public void runDailyAggregationJob(){
        try{
            log.info("runDailyAggregationJob Start!!!");
            String date = LocalDate.now().toString();
            JobParameters jobParameters = new JobParametersBuilder()
                    .addString("date", date)
                    .toJobParameters();
            jobLauncher.run(dailyAggregationJob,jobParameters);
            log.info("runDailyAggregationJob End!!!");
        }catch(Exception e){
            throw new RuntimeException("runDailyAggregationJob 실행 중 오류 발생",e);
        }
    }
}
```

- JobLauncher : `Job`을 실행하는 역할, `Job`과 `JobParameters`를 전달받아 실행
- @Scheduled(cron = "*(초) *(분) *(시) *(일) *(월) *(요일)") : Spring에서 주기적으로 메서드를 실행 할 때 사용하는 설정
    - Spring에서 사용되는 cron 표현식은 **6자리**로 구성
    - 예를 들어, 매주 월요일 새벽 4시 30분 59초 ⇒ @Scheduled(cron = "59 30 4 * * MON")
- JobParameter
    
    ```java
    String date = LocalDate.now().toString();
    JobParameters jobParameters = new JobParametersBuilder()
            .addString("date", date)
            .toJobParameters();
    jobLauncher.run(myJob,jobParameters);
    ```
    
    - 배치를 실행하는 날짜를 `date`로 바인딩하고, 그 값을 “date”라는 이름으로 `JobParameter`를 생성
    - `toJobParameters()`를 이용해 `JobParameter`를 묶은 `JobParameters`를 `jobLauncher`에 전달

## 2. Job 수정

```java

@Slf4j
@RequiredArgsConstructor
@Configuration
public class DailyAggregationConfig {

    private final EntityManagerFactory entityManagerFactory;
    private final int CHUNK_SIZE = 500;

    @Bean
    public Job dailyAggregationJob(JobRepository jobRepository, PlatformTransactionManager transactionManager){
        return new JobBuilder("dailyAggregationJob", jobRepository)
                .start(dailyAggregationStep(jobRepository,transactionManager))
                .build();
    }

    @Bean
    public Step dailyAggregationStep(JobRepository jobRepository, PlatformTransactionManager transactionManager){
        return new StepBuilder("dailyAggregationStep", jobRepository)
                .<Product, Product>chunk(CHUNK_SIZE, transactionManager)
                .reader(dailyAggregationReader())
                .writer(dailyAggregationWriter())
                .build();
    }

    @Bean
    public JpaCursorItemReader<Product> dailyAggregationReader() {

        JpaCursorItemReader<Product> reader = new JpaCursorItemReader<>();
        reader.setName("DailyAggregationReader");
        reader.setEntityManagerFactory(entityManagerFactory);
        reader.setQueryString("select p from Product p");

        return reader;
    }

    @Bean
    public ItemWriter<Product> dailyAggregationWriter() {
        return item->{
            log.info("item => {}",item);
        };
    }

}
```

- EntityManagerFactory : JPA의 핵심 객체 중 하나로, DB와의 실제 상호작용을 담당하는 `EntityManager` 를 생성하는 팩토리
- processor는 필요하지 않아서 제거
- `JpaCursorItemReader` : DB에서 `Product` 엔티티를 **Cursor 방식**으로 읽어옴

## 3. 실행

![image.png](/assets/post_img/250506/image1.png)

![image.png](/assets/post_img/250506/image2.png)

```text
Job: [SimpleJob: [name=dailyAggregationJob]] completed 
	with the following parameters: 
		[{'date':'{value=2025-05-06, type=class java.lang.String, identifying=true}'}] 
	and the following status: [COMPLETED] in 278ms
```

실행 후 로그를 살펴보면 [COMPLETED] 가 출력되었다는 것은 해당 배치 작업이 **정상적으로 완료되었고**, Spring Batch가 **메타 테이블(`BATCH_JOB_INSTANCE`, `BATCH_JOB_EXECUTION` 등)**에 `COMPLETED` 상태로 저장했다는 것을 의미함

그럼 이제 정확히 어떻게 메타테이블에 저장이 되었는지 확인해보자

# 메타테이블

## BATCH_JOB_INSTANCE

![image.png](/assets/post_img/250506/image3.png)

- `JOB_INSTANCE_ID` : `BATCH_JOB_INSTANCE` 의 PK
- `JOB_NAME` : 수행한 Batch Job Name

방금 실행한 dailyAggregationJob이 있는 것을 확인할 수 있고 이 인스턴스는 JobParameter에 따라 생성된다.

확인해보자!

### 1. JobParameter를 수정하지않고 그대로 실행 (’date’ : ‘2025-05-06’)

![image.png](/assets/post_img/250506/image4.png)

오류 발생!

### 2. JobParameter를 수정하고 실행 (’date’ : ‘2025-05-05’)

![image.png](/assets/post_img/250506/image5.png)

작업 성공!

![BATCH_JOB_INSTANCE](/assets/post_img/250506/image6.png)

BATCH_JOB_INSTANCE

![각 Execution에 해당하는 parameter](/assets/post_img/250506/image7.png)

각 Execution에 해당하는 parameter

⇒ **Spring Batch의 중복 방지 기능**

그렇다면 만약 **작업이 실패한 경우 같은 파라미터로 재실행 할 때는??**

### 3. Job 실행 실패 후 재시도

- date : 2025-05-07
- `ItemWriter` 수정 후 실행
    
    ```java
    @Bean
    public ItemWriter<Product> dailyAggregationWriter() {
        return items->{
            for (Product item : items) {
                if (item.getPrice() == null) {
                    throw new IllegalArgumentException("배치 실행 중 null 값 오류 발생!!!");
                }
                log.info("item => {}", item);
            }
        };
    }
    ```

- `ItemWriter` 수정 후 실행

    ```java
    @Bean
    public ItemWriter<Product> dailyAggregationWriter() {
        return items->{
            for (Product item : items) {
                if (item.getPrice() == null) {
                    throw new IllegalArgumentException("배치 실행 중 null 값 오류 발생!!!");
                }
                log.info("item => {}", item);
            }
        };
    }
    ```

  *오류 발생*

  ![image.png](/assets/post_img/250506/image8.png)

  **BATCH_JOB_EXECUTION 테이블에서 FAILED된 것을 확인**

  ![image.png](/assets/post_img/250506/image9.png)


  이제 if문을 주석처리 하고 다시 실행한다면?    

  *작업 성공*

  ![image.png](/assets/post_img/250506/image10.png)
  
  오류 없이 잘 실행이 되고 **BATCH_JOB_EXECUTION** 테이블에서 COMPLETED된 것을 확인
  
  ![image.png](/assets/post_img/250506/image11.png)
  
  **BATCH_JOB_EXECUTION_PARAMS** 테이블에서 **오류로 인해 작업이 중지된다면 같은 파라미터라도 작업을 재시도 할 수 있다**는 것을 알 수 있음
  
  ![image.png](/assets/post_img/250506/image12.png)
  
  그런데 **BATCH_JOB_INSTANCE** 테이블에서 2025-05-07에 대한 Job인스턴스는 하나밖에 없음
  
  ![image.png](/assets/post_img/250506/image13.png)

## **JOB, JOB_INSTANCE, JOB_EXECUTION**

### 용어 정의 및 관계 정리

| 용어 | 설명 |
| --- | --- |
| **Job** | 배치 작업 정의 |
| **JobInstance** | 특정 Job + **JobParameter 조합**에 따라 생성된 인스턴스. 동일 JobParameter로는 **한 번만 생성**됨. |
| **JobExecution** | JobInstance의 실행 기록. 실패했든 성공했든 **실행 시마다 생성**됨. 동일 파라미터로 재시도 가능. |

---

### 테이블 관계 요약

### 1. `BATCH_JOB_INSTANCE`

- 하나의 `JobInstance`는 고유한 `JobName + JobParameter` 조합으로 결정됨.
- 동일 파라미터로 실행하려 하면 **중복으로 간주되어 오류 발생**.
- 컬럼 주요 정보:
  - `JOB_INSTANCE_ID`: PK
  - `JOB_NAME`: Job 이름
  - `JOB_KEY`: JobParameter를 해시 처리한 값 (중복 방지 기준)

### 2. `BATCH_JOB_EXECUTION`

- 하나의 `JobInstance`는 여러 개의 `JobExecution`을 가질 수 있음 (실행마다 새로 생김).
- 실패 후 재시도하면 같은 `JOB_INSTANCE_ID`를 참조하는 새로운 `JobExecution` 생성.
- 상태는 `STARTING`, `FAILED`, `COMPLETED`, `ABANDONED` 등.

### 3. `BATCH_JOB_EXECUTION_PARAMS`

- 해당 `JobExecution`에 사용된 실제 `JobParameter` 값을 저장.
- `JobInstance`에는 파라미터 해시만 저장되므로, 실제 파라미터는 이 테이블에서 확인함.
