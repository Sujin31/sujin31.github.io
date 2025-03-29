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

 Layered Architecture 기반

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

## ENTITY, DTO, VO

1. 사용자가 입력한 데이터 VO형태로 저장
2. 데이터를 받아 from 메소드를 통해 DTO로 변환
3. toEntity메소드를 사용해 Entity로 변환하고 DB저장
