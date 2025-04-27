---
title: "RDB 기초 및 데이터베이스 설계"
date: 2025-03-15
description: "RDB 기본 개념 및 설계 원칙 정리"
categories:
  - Workshop
  - 1차 Project
tags: 
  - RDB
  - 설계
---

# RDB 기초 및 데이터베이스 설계

## 1. 이름 작성 원칙

테이블 및 엔티티의 이름은 풀네임(Full Name) 으로 작성하고, 단수형(Singular Form) 을 사용한다.

### 예시:

- User (❌ Users X)
- Product (❌ Products X)
- Order (❌ Orders X)

## 2. 하이버네이트(JPA) 활용

JPA(Hibernate)를 사용하면 클래스를 먼저 만들고, 이를 기반으로 데이터베이스 테이블을 자동 생성할 수 있다.

## 3. PK(기본 키) 설정 원칙

PK는 유효 아이디(예: 주민등록번호, 이메일)를 사용하지 않는다.

검색 성능이 저하될 수 있기 때문.

대신 Long 타입의 숫자를 PK로 사용해야 한다. (int는 범위 제한이 있음.)

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

## 4. PK는 "테이블 정의"가 아니다

PK는 단순히 "이 행이 유일하다"는 의미일 뿐, 테이블의 구조를 결정하는 것이 아님.

## 5. 메인이 되는 객체(중심 엔티티)에는 FK를 직접 사용하지 않는다

핵심 개체(예: User, Product)는 직접 FK(외래 키)를 두지 않는 것이 좋다.

### 이유:

- FK를 직접 두면 조회 시 불필요한 데이터까지 로딩되어 성능 저하.
- 유지보수 측면에서 결합도가 높아져 의존성이 증가.

```java
public class Product {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category; // ❌ 직접 FK를 가지면 불필요한 로딩 발생
}
```

## 6. 객체는 FK를 직접 가지지 않고, 중간 테이블을 통해 관리한다

N:M(다대다) 관계에서는 중간 테이블을 만들어서 FK를 설정하는 것이 좋다.

### 예시: 상품(Product)과 카테고리(Category)의 관계

```java
@Entity
public class Product {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class Category {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
}

@Entity
public class ProductCategory {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "product_id")
    private Product product;

    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category;
}
```

## 정리

- ✅ 이름은 단수형으로 작성 (User, Product)
- ✅ JPA를 활용하면 클래스 기반으로 자동 테이블 생성 가능
- ✅ PK는 Long 타입 숫자로 설정 (유효한 아이디 사용 X)
- ✅ PK는 테이블 정의가 아니라 개별 행을 구분하는 역할
- ✅ 중심 객체에는 FK를 직접 두지 않음 (불필요한 데이터 로딩 방지)
- ✅ N:M 관계에서는 중간 테이블을 생성해서 FK를 관리

