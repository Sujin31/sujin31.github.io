---
title: "HTTP MESSAGE"
date: 2025-03-23
description: "HTTP > HTTP 메시지"
categories: 
  - Cs
  - Web
tags: 
  - HTTP
  - HTTP MESSAGE
  - CS
  - WEB
---
# Request Message

HTTP Request Message는 공백을 제외하고 3가지 부분으로 나뉨

- Start Line
- Headers
- Body

![http_header.png](/assets/post_img/250323/http_m_header.png)

## **Start Line**

HTTP Request Message의 시작 라인이며 `HTTP method`, `Request target`, `HTTP version` 으로 구성

```html
[HTTP method]  [Request target] [HTTP version]
GET /test.html HTTP/1.1
```

### HTTP METHOD

- 종류
    - GET : 리소스 조회
    - POST : 요청 데이터 처리, 주로 등록에 사용
    - PUT : 리소스를 대체, 해당 리소스가 없다면 생성
    - PATCH : 리소스 부분 변경
    - DELETE : 리소스 삭제
    
- 속성
    - 안전 : 여러 번 호출해도 리소스의 상태를 변경하지 않음
    - 멱등 : f(f(x)) = f(x),  번 호출하든 두 번 호출하든 100번 호출하든 결과가 똑같음
        - 활용 : 자동 복구 메커니즘
    - 캐시가능 : 응답 결과 리소스를 캐시해서 사용해도 되는가?

![image.png](/assets/post_img/250323/http_m_image.png)


<details markdown="1">
<summary>put은 왜 멱등하고 patch는 왜 멱등하지않지?</summary>

- “멱등은 같은 요청을 여러번 보내더라도, 결과는 항상 동일해야한다.”
    
    **put 의 경우**
    
    - **리소스 전체를 덮어쓰는 방식**이기 때문에 동일한 요청을 여러번 보내도 리소스의 최종 상태는 변하지 않음
    - patch처럼 일부만 연산할 수 있지 않나? ⇒ 기존 데이터를 덮어쓰기 때문에 연산이 작동하지않음
    
    **patch 의 경우** 
    
    ```html
    PATCH /account/123 HTTP/1.1
    Host: example.com
    Content-Type: application/json
    
    {
      "balance": "+100"
    }
    ```
    
    - 기존값에 100을 더하는 연산 방식으로 인해 결과가 항상 달라짐
    - patch도 멱등하게 동작할 수 도 있지만 일반적으로 **멱등하지 않다**고 간주함
</details>

---

## HEADERS

- 해당 Request에 대한 추가 정보를 담고있는 부분
- 필요 시 임의의 헤더 추가 가능

<aside class="notion_callout">

[field-name]: [field-value]

```html
Host: google.com
Accept: text/html
Accept-Encoding: gzip, deflate
Connection: keep-alive
MemberUuid: ABC
```

</aside>

---

## BODY

- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등 byte로 표현할 수 있는 모든 데이터 전송 가능
- 전송할 데이터가 없는경우 비어있

```html
{
    "test_id": "tmp_1234567",
    "order_id": "8237352"
}

OR

<html>
 <body>...</body>
</html>
```

---

# Response Message

HTTP Response Message는 공백을 제외하고 3가지 부분으로 나뉨(Request와 동일)

- Status Line
- Headers
- Body

![image.png](/assets/post_img/250323/http_m_image1.png)

### Status Line

HTTP Response의 상태를 간략하게 나타내주는 부분

<aside class="notion_callout">

[HTTP-version] [status-code] [reason-phrase]

HTTP/1.1 200 OK

</aside>

- HTTP 버전
- HTTP 상태 코드 : 요청 성공, 실패를 나타냄
    - 200 : 성공
    - 400 : 클라이언트 오류
    - 500 : 서버 내부 오류
- 이유 문구 : 사람이 이해할 수 있는 짧은 상태 코드 설명 글



이하 동일..


[HTTP 로 돌아가기](/sujin31/posts/HTTP) 
