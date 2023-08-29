---
layout: post
title:  "HTTP Header를 왜 사용할까?"
date:  2022-06-21 00:00:00 +0900
categories: cs
tags: cs
---

## HTTP header?

HTTTP에 왜 header가 있을까? 그리고 이 header를 어떤 용도로 사용할까? header에 있는 내용을 body에 넣어도 문제없지 않을까?  

## 왜 사용할까?

TCP/UDP와 같은 packet 통신에서는 header 정보를 사용하여 packet의 종류, 길이, 체크섬 등 패킷을 검증하고 처리하는데 사용한다. HTTP header도 마찬가지로 Http request/response에 대한 처리를 위해 부가적인 정보들을 담아 이를 처리하기 위해 사용한다. 각 사이드(server 혹은 client)의 상태 정보, 동작 정보 및 전송되는 데이터의 명세 정보를 포함한다. 예를 들어, Content-Type:application/json header는 해당 http 메소드에 포함된 컨텐츠의 타입이 json 형태임을 명시하며, 서버에서 이 request에 대한 body를 처리할 때 데이터 파싱을 json을 기반으로 진행하게 된다.

## HTTP header의 spec

- 모든 header는 case-intensive(대소문자 구별이 없음)하다.
- header field는 콜론(:)으로 구분된다.
- key-value로 이루어진 문자열을 받는다.

HTTP header의 각 필드들의 목록 및 의미에 대해서는 [Mozilla](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers)의 페이지를 참조하자.

## Preflight?

Client에서 HTTP를 통해 server로 요청을 보낼 때, 이 요청에 대한 검증을 진행한다. 이 때, HTTP header에는 Access-Control-Request-Method, Access-Control-Request-Headers, Origin을 담아 server에서 이를 확인하도록 한다. 이를 통해 client는 server가 이 api를 처리할 수 있는지 빠르게 알 수 있으며, 불필요하게 큰 (post/put 요청 등의)body를 서버로 보내 통신 비용의 낭비를 줄인다. 또한 CORS를 통해 다른 domain의 요청을 차단하거나 허용하는 등의 보안 조치도 가능하다.

## Reference

- [Mozilla: HTTP 헤더](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers)
- [Mozilla: HTTP Preflight](https://developer.mozilla.org/ko/docs/Glossary/Preflight_request)
- [Mozilla: CORS](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)