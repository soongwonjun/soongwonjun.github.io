---
layout: post
title:  "nameserver에 대해서"
date:  2022-12-10 00:00:00 +0900
categories: cs
tags: linux cs network
---

## nameserver란?

DNS(Domain Name System) 서버를 구성하는 하나하나의 서버이다. nameserver는 DNS를 탑재하고 있으며, 이러한 nameserver가 모여 DNS를 만든다.  
각각의 nameserver는 domain과 ip의 매핑 정보 혹은 domain에 이어질 다른 nameserver 주소 정보를 가지고 있다.

## 내가 원하는 domain까지 이르는 과정

우리는 특정 사이트를 접속할 때 domain을 주로 사용한다. google.com과 같은 domain이 여기에 속한다.  
PC에서는 이 domain을 ip주소로 바꾸고, 그 ip주소에 해당하는 서버에 컨텐츠를 요청한다.
이 때 이 domain과 ip 혹은 새 domain 매핑하고, 여기서 매핑되는 ip 주소를 a-record(address-record), 도메인을 c-name(canonical name)라고 부른다.

- a-record는 해당 웹 사이트의 ip주소이다.
- c-name은 해당 domain 혹은 subdomain의 정보를 가진 다른 nameserver의 domain이다
이 과정은 a-record를 얻거나, 최종적으로 domain ip를 얻을 수 없을 때 까지 반복되며, ip를 얻게 되면 해당 서버에 컨텐츠를 요청하고 그렇지 않으면 에러를 표기한다.

## Linux 환경에서 nameserver 적용하기

- /etc/resolv.conf 수정하기
- shell에서 network 재시작하기

```terminal
# service를 사용한 재시작
service network restart

# NetworkManager를 사용한 재시작
systemctl stop NetworkManager.service
systemctl disable NetworkManager.service

systemctl start NetworkManager.service
systemctl enable NetworkManager.service
```
