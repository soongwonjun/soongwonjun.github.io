---
layout: post
title:  "Linux의 Standard I/O Stream의 Cheatsheet"
date:  2022-03-14 12:00:00 +0900
categories: commands
tags: linux cheatsheet
---

## Linux Standard I/O Stream

- 0: STDIN
- 1: STDOUT
- 2: STDERR

## Redirect

- command 2>&1 : STDERR를 STDOUT으로 redirect하여 STDOUT을 출력한다.
- command 2>&1 > out.log : STDERR를 STDOUT으로 redirect하여 STDOUT을 out.log에 저장한다.
- command &> : 2>&1과 동일
