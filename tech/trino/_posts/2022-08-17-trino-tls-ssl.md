---
layout: post
author: author1
title: Trino TLS/SSL 적용하기
categories: [trino]
description: >
  
---

# Trino TLS/SSL 적용하기

## 1. Trino Security

기본 설정으로 Trino를 설치해보면 로그인 과정은 있지만 id는 아무렇게나 입력해도되고 password는 입력받지 않는다.  
그러므로 보안 설정을 적용해보자.  
여러 방법이 있지만 간단한 방법 중 하나인 `Password file authentication`으로 진행을 해본다.  

하지만 `Password file authentication`을 적용하기 위해선 두 가지선행 작업이 필요하다.  
> Using TLS and a configured shared secret is required for password file authentication.

그것들을 먼저 진행해본다.

## 2. TLS 설정

인증서를 발급받아야 하는데 내부망 개발 환경이므로 `self-signed certificates`를 통해 구성해본다.

#### 2.1. ROOT 인증서 생성

* 디렉토리 생성

~~~shell
[igkim@trino ~]$ mkdir rootca
[igkim@trino ~]$ mkdir trino
~~~

* ROOT CA RSA Key 생성

~~~shell
[igkim@trino ~]$ cd rootca
[igkim@trino rootca]$ openssl genrsa -aes256 -out rootca.key 2048

Generating RSA private key, 2048 bit long modulus (2 primes)
....+++++
.................................................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for rootca.key: {Root RSA Key Password 입력}
Verifying - Enter pass phrase for rootca.key: {Root RSA Key Password 입력}
~~~

* ROOT CSR Config 작성

~~~shell
[igkim@trino rootca]$ vi rootca_openssl.conf
~~~


## 3. Configured shared secret



## 4. Password file authentication





