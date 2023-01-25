---
title: AES, RSA, SHA-256 암호화의 차이점
layout: single
categories: theory
tag: ['개발이론', '백엔드', '암호화']
toc: true
toc_label: "목차"
toc_icon: "fas fa-list-ul"
author_profile: false
sidebar:
    nav: "docs"
---

## AES 알고리즘

- **AES는** 암/복호화 과정에서 동일한 키를 사용하는 **대칭 키 알고리즘** 이다.
- **AES** 암/복호화 키는 절대로 외부에 유출되지 않도록 관리해야 하여야 한다. **비밀키(Secret Key)**
- 가변 길이의 블록과 가변 길이의 키 사용이 가능하다.
- AES-128, AES-192, AES-256(키의 길이 16byte, 24byte, 32byte)
  128bit 키 사용 시에는 10라운드, 192bit에서는 12라운드, 256bit에서는 14라운드를 실행

## RSA 알고리즘

- **RSA**는 암/복호화 과정에서 서로 다른키를 사용하는 **비대칭 키 알고리즘** 이다.
- RSA 암/복호화 키는 **공개키(Public Key)**와 **개인키(Private Key)**로 구성된다.
- **공개키(Public Key)**는 암호화할 때 사용하고, **개인키(Private Key)**는 복호화할 때 사용한다.

## SHA-256

- **SHA-256**은 **SHA**(Secure Hash Algorithm) 알고리즘의 한 종류로서 256비트로 구성되며 64자리 문자열을 반환된다.
- **SHA-256** 해시 함수는 어떤 길이의 값을 입력 하더라도 256비트의 고정된 결과값을 반환한다.
- **SHA-256**은 단방향 암호화 방식이기 때문에 복호화가 불가하다.

