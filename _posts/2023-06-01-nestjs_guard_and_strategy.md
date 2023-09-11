---
layout: post
title:  NestJS의 Guard의 작동 방식
date:  2023-06-01 22:00:00 +0900
categories: nestjs
tags: nodejs nestjs
---

## NestJS Guard

![image](https://docs.nestjs.com/assets/Guards_1.png)

guard에서는 `canActivate`로 이 요청이 처리될 수 있음을 결정한다.
authguard와 passportstrategy는 같이 쓰이게 되며, 여기에서 guard.canActivate -> strategy.isValidate 를 호출해서 처리한다.
따라서 `controller`에서 주입받은 `request` 객체에 `user` 혹은 guard에서 지정한 필드(constructor의 `option.property`)가 없다면 이를 한번 확인을 해 두어야 할 것이다.

### 보충

- passport의 authGuard는 canActivate 시 context validation을 위한 strategy를 생성하는데 이 strategy가 생성 될 때 callback으로 항상 validation을 실행할 수 있도록 한다.

## References & Reference

[Manual: Guards](https://docs.nestjs.com/guards)
