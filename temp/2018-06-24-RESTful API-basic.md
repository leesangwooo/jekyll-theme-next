---
title: RESTful API의 정의
description: RESTful API의 정의
categories:
 - labs
tags: rest
---

## REST
REpresentational State Transfer

: 서버와 클라이언트가 리소스를 교환하기 위해 리소스에 식별자를 부여하는 규칙, 스타일 

>  참고.[RFC3986(영문)](http://www.rfc-editor.org/rfc/rfc3986.txt)

- 리소스를 유일한 URI(Uniform Resource Identifiers) 형태로 표현한다.
- 리소스의 상태는 HTTP Methods를 활용해서 구분한다. 
xml 또는 json 포맷으로 데이터를 전송한다. 

## do / don't
1. URI의 표현
    - Do 명사형
    - Don't 동사형
     > 동사적인 표현을 URI에 포함하지 않습니다. 대신, HTTP Method로 action을 구분합니다.

2.       




## CRUD

| action | http method |
|:---:|:---:|
| Create | POST | 
| Retrieve | GET |
| Update | PUT |
| Delete | DELETE |
