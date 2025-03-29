---
title: "JPA - UPDATE"
date: 2025-03-29
description: "JPA - UPDATE 하는 방식 3가지"
categories:
  - JPA
tags:
  - JPA
  - update
  - delete
  - persistence
  - dirty checking
  - bulk operation
---

# 1. DTO → Entity 감싸서 save

# 2. Dirty Checking

- JPA가 제공하는 기능으로, **엔티티 객체**의 **상태가 변경되었는지 자동으로 감지**하고 변경된 사항을 **DB에 반영**함
- 즉, **영속성 컨텍스트**에 있는 엔티티 객체가 변경되었을 때, 이를 자동으로 감지하고 **flush**할 때 해당 변경 사항을 DB에 반영
- 사용 시 주의할점
  - setter를 직접 사용하는 대신 **변경작업에 의미를 부여하는 메서드명 사용**
  - 영속성이 깨질 수 있음
  - JPA에 종속되어 있는 기능임(강결합) → infrastructure(ex.어댑터아키텍쳐)가 변경된다면 안스는게 좋음(유연성)

# 3. JPQL

- 부분 변경 + **대용량 처리**
- `@Query` 또는 `@Modifying`을 사용
- 벌크 연산 → **영속성 주의**
  - 여러 개의 데이터를 한 번에 처리하는 연산
  - JPA에서 벌크 연산은 `@Query`, `@Modifying` 어노테이션과 함께 JPQL을 사용하여 다량의 데이터를 한 번에 업데이트하거나 삭제하는 작업
  - **문제점 : 벌크 연산 실행 시 영속성 컨텍스트를 무시하고 DB에 직접 적용됨**
  - **해결방안**
    - **벌크 연산 전에 `flush()` 필요 → `flushAutomatically = true` 사용**
    - **벌크 연산 후 `clear()` 필요 → `clearAutomatically = true` 사용**
    - **벌크 연산 후 `refresh()`는 잘 사용되지 않으며, 대신 `clear()` 후 다시 조회하는 것이 일반적**
    - **가능하면 벌크 연산 후 `findById()` 등을 이용해 필요한 데이터만 다시 조회**
