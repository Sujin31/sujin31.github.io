---
title: "2차 프로젝트 - READ TABLE 설계"
date: 2025-05-28
description: "2차 프로젝트 공모 서비스 개발 중 READ 데이터 생성하는데 생긴 궁금증과 문제점 해결"
categories:
  - Workshop
  - 2차 Project
tags:
  - Workshop
  - Project
  - CQRS
  - MSA
  - EDA
---

## 개요

- **MSA** 도입과 **DDD** 적용으로 상품 서비스와 공모 서비스를 분리함
- **DB 설계(RDB**) : 상품 테이블, 공모 테이블

![공모와 상품 테이블은 연관관계 (FK 설정은 안함 ⇒ MSA)](/assets/post_img/250528/image.png)
공모와 상품 테이블은 연관관계 (FK 설정은 안함 ⇒ MSA)

- 로직 상 관리자가 상품을 등록 후 공모 상품을 등록 할 수 있음
- (CQRS 적용) 사용자에게 보여줘야할 페이지에서 상품과 공모 테이블 뿐만아니라 이미지, 카테고리 등 다양한 테이블을 조인해 가져와야하기때문에 READ 테이블 설계를 시도함
    
    ```json
    {
      "productUuid": "string",
      "productName": "string",
      "productCategoryId" : "number",
      "mainCategoryName" : "string",
      "subCategoryName" : "string",
      "aiEstimatedPrice": "number",
      "purchasePrice": "number",
      "productStatus": "string",         
      "storageLocation": "string",
      "description": "string",
      "images": [                          
        {
          "imageUrl": "string",
          "imageIndex": "number",
          "isThumbnail": "boolean"
        }
      ],
      "fundings": [                        
        {
          "fundingUuid": "string",
          "fundingAmount": "number",
          "piecePrice": "number",
          "totalPieces": "number",
          "remainingPieces": "number",
          "fundingDeadline": "date",
          "fundingStatus": "string"  ,
          "isDeleted": "boolean"
        }
      ]
    }
    
    ```
    

---

## 문제점

> READ 데이터는 어디서 언제 만들어야하나

- 상품 서비스와 공모 서비스가 분리되어 있어, 상품 등록 후 공모 등록이 완료됐을 때 두 서비스를 결합하여 READ 데이터를 생성하는 과정에서 **서비스 간 결합도 증가 문제 발생** ⇒ 서비스 분리한 이유가 없어짐

- 상품 등록 직후는 공모 데이터가 없으므로 상품 데이터만 우선 저장하고, 공모 등록 시점에 READ 데이터를 업데이트하는 방식을 선택함 → 이 또한 데이터 동기화와 복잡도 문제 존재

---  

## 해결방안: 이벤트 기반 아키텍처(EDA) 활용

- **상품 등록/수정, 공모 등록/수정 시 이벤트 발행**
    
    각 서비스는 자신의 상태 변경 시점에 관련 이벤트를 메시지 브로커(Kafka, RabbitMQ 등)로 발행
    
    예) `ProductCreated`, `FundingCreated` 등 이벤트
    
- **READ 데이터 전용 서비스에서 이벤트 구독 및 처리**
    
    별도의 READ 전용 서비스가 해당 이벤트들을 구독하여,
    
    상품 및 공모 관련 데이터를 조합해 READ용 데이터베이스에 저장하거나 업데이트
    
- **비동기 처리로 서비스 간 결합도 최소화**
    
    상품 서비스와 공모 서비스는 서로 직접 호출하지 않고 이벤트를 통해 간접적으로 통신하므로, 독립성과 확장성이 보장
    

---

MSA, EDA(kafka), CQRS 에 대해서 더 공부해야겠다
