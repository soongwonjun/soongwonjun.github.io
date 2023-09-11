---
layout: post
title:  "HTTPS 통신을 위한 SSL Handshake"
date:  2022-05-26 18:00:00 +0900
categories: cs
tags: cs network
description: ""
---

## SSL Handshake

![SSL_Handshake](/images/20220526/ssl_handshake.gif)

- client, server간의 syn/syn-ack/ack 가 끝난 후에 SSL Handshake가 시작된다.
  1. ClientHello: client는 client hello 와 함께 프로토콜, 사용 가능한 암호화 알고리즘 방식, 키 등의 목록을 chiper suites에 담아 서버로 전송한다.
  2. ServerHello: server는 client hello 와 함께 전송된 cipher suits중 하나를 선택해 client에 알리고 싱크를 맞춘다. 필요에 따라 클라이언트에 인증서를 요청하기도 한다.
  3. client에서 인증서를 검증하고, 사용할 chiper suits를 확인한다. 또한 서버로 전송할 비밀키를 만들어 서버로 전송할 준비를 한다. 이 때 검증 결과는 hash로 만들어지며 이 값으로 SSL/TLS session을 검증할 때 사용한다. 혹시 client나 server의 정보 일부가 변경되면 hash값도 달라지게 되고, 암호화 통신에 문제가 생기니 SSL/TLS session은 끊어진다.
  4. client에서 준비된 인증서를 서버로 전송한다.
  5. 필요에 따라 서버로 클라이언트의 인증서를 전송하며, 이 인증서가 필수로 요구되는 경우에는 이 인증서에 대한 검증을 거치며, 검증이 실패하는 경우에는 SSL Handshake가 실패한다. 이 시점에는 이미 client와 server가 암호화를 위한 키를 교환하였기 때문에 이 때부터 전송되는 데이터는 암호회되어 있다.
  6. 서버는 클라이언트의 인증서를 검증한다.
  7. client는 server로 finished 를 보내어 SSL handshake가 끝났음을 알린다.
  8. server는 client로 finished 를 보내어 SSL handshake가 끝났음을 알린다. 이로서 client와 server간의 SSL/TLS session이 맺어지게 된다.
  9. 이제부터 server와 client는 SSL/TLS session 내에서 암호화된 데이터를 교환한다. SSL/TLS session 내의 데이터는 암호화되어 있어, packet snippet과 같은 작업을 하여도 실제 데이터를 볼 수 없는 상태가 된다.

## Reference & References

- [SSL Handshake](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=ssl-overview-tls-handshake)