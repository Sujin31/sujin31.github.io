---
title: "JPA - 다대다 관계와 중간테이블"
date: 2025-03-29
description: "JPA - 다대다 관계와 중간테이블의 역할"
categories:
  - JPA
tags:
  - JPA
  - N:M
  - MiddleTable
---
# JPA - 중간테이블

중간테이블(middle table)

# 다대다[N:M]

## RDB

- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
- 연결 테이블(조인 테이블)을 추가해서 일대다, 다대일 관계로 풀어야함
    
    ex) 상품 - 옵션 관계 정의 방법
    
    1. 옵션을 ,(콤마)로 나눠서 List로 넣는다 (s,빨강, …) ⇒ X
    2. 옵션 테이블 설계 : 상품은 여러개 옵션을 가지게 됨 ⇒ 같은 옵션 중복 문제
    3. **중간 테이블 추가**
    
    ```sql
    CREATE TABLE Product(
        id INT PRIMARY KEY,
        name VARCHAR(100)
    );
    
    CREATE TABLE Option(
        id INT PRIMARY KEY,
        name VARCHAR(100)
    );
    
    -- 연결 테이블 (중간 테이블)
    CREATE TABLE Product_Option (
        product_id INT,
        option_id INT,
        PRIMARY KEY (product_id , option_id ),
        FOREIGN KEY (product_id ) REFERENCES Product(id),
        FOREIGN KEY (option_id ) REFERENCES Option(id)
    );
    ```
    
    ### **🔹 중간 테이블의 역할**
    
    ✔ **1:N + N:1 관계로 변환하여 다대다를 해결**
    
    ✔ **중복 데이터를 방지하여 정규화된 데이터 구조 유지**
    
    ✔ **추가적인 속성을 포함할 수 있음 (예: 등록 날짜, 점수 등)**
    
    ✔ **키 값만 존재해서 가벼움(빠르다, 유연함)**
    
    ✔ **기능테이블**
    
    - manyToOne , oneToMany (X) ⇒ N+1 문제
    

---

## ORM

- 컬렉션을 사용해 객체 2개로 다대다 관계가 가능 (List, Set 등)
- 단방향 다대다 관계

```java
@Entity
public class Member {
  ... 
	 @ManyToMany 
	 @JoinTable(name = "member_product")//연결 테이블 지정 가
	 private List<Product> products = new ArrayList<>();
  ...
}

```

---

## ORM에서 다대다 관계가 문제가 되는 경우

### 1. 관계형 DB와 객체 모델의 불일치

- RDB에서는 항상 중간 테이블을 사용해야 하지만, 객체에서는 컬렉션 만으로 다대다 표현 가능

### 2. 다대다 관계에서는 추가적인 정보(속성)을 넣기 어려움

- 만약 **Member_Product** 테이블에 `등록일`, `수량` 같은 속성을 추가해야한다면?
- 객체에서는 직접 표현이 어려움 → 결국 엔티티를 만들어야 함

### 3. 성능 이슈

- JOIN이 많이 발생해 성능이 저하될 수 있음

---

## 다대다 한계 극복

**중간 엔티티를 직접 만들어서 사용**

```sql
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}

@Entity
public class Product {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}

@Entity
public class MemberProduct {  -- 중간 엔티티 직접 생성
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Member member;

    @ManyToOne
    private Product product;

    private LocalDateTime createdAt; -- 추가 속성
}

```

⇒  `@OneToMany` 사용 시 N+1문제 생길 수 잇음

---

# 기타

⚔️장바구니 삭제 한다?

→ 빅데이터를 위해 장바구니 삭제해도(update) 진짜 삭제 X → 상태만 false로 변경 (soft delete)

⚔️객체 생성 후 바로 릴레이션?

→ middle table 설계부터 확인!!!!

⚔️하나의 함수는 하나의 기술만 (테스트코드 안해도됨) ⇒ Solid 원칙

⚔️ JOIN을 최소화

⚔️ for 보다는 Map or String 사용
 
 
 
 
 
<details markdown="1">
<summary>참고</summary>    

- [https://ict-nroo.tistory.com/127](https://ict-nroo.tistory.com/127)

</details>
