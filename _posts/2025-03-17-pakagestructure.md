---
title: "패키지구조 정리"
date: 2025-03-15
description: "실습 패키지구조 정리"
categories: 
  - Workshop
  - package structure
tags: 
  - package
  - structure
  - entity
  - dto
  - vo
---
# 패키지구조 정리

스킬업 BACKEND 실습 패키지 구조

## 🎯 **Spring Boot 패키지 구조 설명**

Spring Boot 프로젝트의 패키지 구조는 **책임 분리**와 **유지보수성**을 고려하여 설계합니다. 대표적인 패키지 구조는 **애플리케이션의 역할에 맞게 계층을 나누는 방식**입니다.

### 1. **`application`** (애플리케이션 계층)

- 비즈니스 로직을 조합하고 유스케이스를 처리합니다.
- **서비스(Service)** 클래스들이 위치하며, 도메인 객체를 활용해 실제 비즈니스 흐름을 만듭니다.

### 2. **`domain`** (도메인 계층)

- 핵심 비즈니스 로직이 들어갑니다.
- **엔티티(Entity)**, **도메인 서비스**, **리포지토리 인터페이스** 등이 위치합니다.

### 3. **`dto`** (Data Transfer Object)

- 계층 간 데이터 전달을 위한 객체입니다.
- 클라이언트와 서버, 또는 서버 간 데이터 전송 시 사용됩니다.

### 4. **`infrastructure`** (인프라 계층)

- 데이터베이스, 외부 API, 파일 시스템과의 연결을 담당하는 계층입니다.
- **JPA 리포지토리**, **API 클라이언트**, **메시징 시스템** 등이 이곳에 위치합니다.

### 5. **`presentation`** (프레젠테이션 계층)

- 사용자와의 상호작용을 처리하는 계층입니다.
- **Controller**가 위치하며, HTTP 요청을 받고 응답을 반환합니다.

### 6. **`vo`** (Value Object, 값 객체)

- **불변(immutable)** 객체로 값으로만 구별되는 객체입니다.
- 예를 들어, `Address`, `Money`, `PhoneNumber` 등이 여기에 해당합니다.

---

## 🚀 **클린 아키텍처 (Clean Architecture)**

### ✅ **개념**

- *Robert C. Martin (Uncle Bob)**이 제안한 아키텍처 패턴입니다.
- **비즈니스 로직**(도메인)을 중심으로, 의존성은 **안쪽**으로만 향하도록 설계합니다.
- 외부 기술(프레임워크, DB, 외부 API 등)에 의존하지 않고, **핵심 비즈니스 로직은 독립적**으로 유지됩니다.

### ✅ **클린 아키텍처의 계층 구조**

```

+-----------------------+
|   Presentation        | → 사용자와 상호작용
+-----------------------+
|   Application         | → 유스케이스를 처리
+-----------------------+
|   Domain              | → 핵심 비즈니스 로직
+-----------------------+
|   Infrastructure      | → DB, 외부 시스템과의 연결
+-----------------------+

```

- **Domain**: 핵심 비즈니스 로직(엔티티, 도메인 서비스 등)
- **Application**: 도메인 로직을 활용해 유스케이스 처리
- **Presentation**: HTTP 요청 처리
- **Infrastructure**: DB, 외부 API 연결 등 외부 시스템 연동

### ✅ **의존성 규칙 (Dependency Rule)**

- **의존성은 항상 안쪽으로만 향해야 한다.**
- `Domain` → 아무것도 의존하지 않음
- `Application` → `Domain`만 의존
- `Presentation` → `Application`만 의존
- `Infrastructure` → 외부 시스템과 연결

### ✅ **장점**

- **유지보수성**: 외부 기술에 영향을 받지 않으므로 변경이 쉬움
- **확장성**: 새로운 요구사항을 추가할 때, 각 계층만 확장하면 됨
- **테스트 용이성**: 각 계층이 독립적이기 때문에 테스트가 쉬움

---

## 🧩 **DDD (Domain-Driven Design, 도메인 주도 설계)**

### ✅ **개념**

- **Eric Evans**가 제안한 설계 방법론으로, **비즈니스 도메인**을 중심으로 소프트웨어를 설계합니다.
- 도메인 모델을 기반으로 **복잡한 비즈니스 로직을 코드로 명확하게 표현**하는 데 초점을 맞춥니다.

### ✅ **DDD의 핵심 개념**

| 개념 | 설명 |
| --- | --- |
| **Entity** | 고유 식별자를 가진 객체 (예: `Order`, `User`) |
| **Value Object (VO)** | 값으로만 구별되는 객체 (예: `Address`, `Money`) |
| **Aggregate** | 관련된 엔티티와 VO를 하나로 묶은 객체 (예: `Order`) |
| **Aggregate Root** | Aggregate의 진입점, 외부에서 Aggregate를 제어할 때 사용 |
| **Repository** | Aggregate를 영속화(저장)하는 역할 |
| **Service** | 도메인 로직 중, 한 엔티티에 속하지 않는 복잡한 로직 처리 |
| **Factory** | 객체 생성 로직을 관리 (복잡한 생성 과정 캡슐화) |
| **Domain Event** | 도메인에서 발생한 이벤트 (예: 주문 완료 이벤트) |
| **Bounded Context** | 도메인의 경계를 명확히 나눈 것 (예: 주문, 결제, 배송) |
| **Ubiquitous Language** | 개발자와 비즈니스 전문가가 공유하는 언어 |

### ✅ **DDD 예시 (주문 도메인)**

```java
// 엔티티 (Entity)
@Entity
public class Order {
    @Id
    private Long id;
    private OrderStatus status;

    @Embedded
    private Address address;  // 값 객체

    public void completeOrder() {
        if (status != OrderStatus.PAID) throw new IllegalStateException("Payment required");
        this.status = OrderStatus.COMPLETED;
    }
}

// 값 객체 (Value Object)
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;
}

// 리포지토리 (Repository)
public interface OrderRepository extends JpaRepository<Order, Long> {}

// 서비스 (Service)
@Service
public class OrderService {
    private final OrderRepository orderRepository;

    public void completeOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
                .orElseThrow(() -> new IllegalArgumentException("Order not found"));
        order.completeOrder();
    }
}

```

### ✅ **DDD의 핵심 원칙**

1. **도메인에 집중**: 복잡한 비즈니스 로직을 도메인 모델로 명확하게 표현
2. **Bounded Context**: 복잡한 시스템을 작은 도메인 단위로 나누어 설계
3. **Ubiquitous Language**: 개발자와 비즈니스 전문가가 동일한 언어로 소통

### ✅ **DDD의 장점**

- 복잡한 비즈니스 로직을 **명확하게 모델링**
- **유지보수와 확장성 향상**
- **팀 간 소통을 원활하게** 하고, 도메인 모델에 집중하여 핵심 로직을 쉽게 관리

---

### ✅ **클린 아키텍처 vs DDD 차이점**

| 구분 | 클린 아키텍처 | DDD |
| --- | --- | --- |
| **목적** | 시스템의 아키텍처를 계층적으로 설계 | 복잡한 도메인 문제를 모델링 |
| **중심** | 계층적 구조와 의존성 규칙 | 도메인 모델과 비즈니스 로직 |
| **주요 요소** | Domain, Application, Presentation, Infrastructure | Entity, VO, Aggregate, Service, Repository |
| **의존성** | 안쪽으로만 의존 | 도메인 모델은 외부에 의존하지 않음 |
| **관계** | DDD를 기반으로 클린 아키텍처를 구현 가능 | 클린 아키텍처의 Domain 계층에서 적용 가능 |

---

### ✅ **결론**

- **클린 아키텍처**는 시스템의 **구조적 설계**를 다루며, **비즈니스 로직은 외부 기술에 의존하지 않게** 하는 것이 목적입니다.
- **DDD**는 **도메인 모델**을 중심으로 복잡한 비즈니스 로직을 다루며, **도메인에 집중한 설계**입니다.
- 두 가지를 **함께 적용**하면, **잘 설계된 도메인 모델**과 **유지보수성, 확장성**이 뛰어난 시스템을 구축할 수 있습니다!

---

## ENTITY, DTO, VO

1. 사용자가 입력한 데이터 VO형태로 저장
2. 데이터를 받아 from 메소드를 통해 DTO로 변환
3. toEntity메소드를 사용해 Entity로 변환하고 DB저장
