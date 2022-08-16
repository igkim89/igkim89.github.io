---
layout: post
author: author1
title: Trino install
categories: [trino]
description: >
  
---

# Trino install

## 1. Trino?

PrestoSQL로 알려졌던 빅데이터용 분산 SQL 쿼리 엔진  
PrestoSQL에서 Trino로 프로젝트명이 변경되었다.  
Facebook에서 개발된 Presto가 점점 Facebook에 종속되어 간다고 느낀 몇몇 개발진들이 나와서  
완전한 오픈소스로 다시 만들어낸 뭐 그런 느낌  
실제로 PrestoSQL을 만들자마자 Facebook측에서는 Presto 상표권을 신청했고
PrestoSQL이라는 이름을 더 이상 쓸 수 없게 되자 Trino로 이름을 변경했다고 한다.  
뭐 그런 역사가 있고..

이기종 데이터 소스에 동일한 ANSI SQL이 사용 가능하며 퍼포먼스가 좋아서 많이들 사용하는 Trino를 설치해본다.

## 2. Trino cluster install

On-premise 환경의 Rocky 8 OS에 설치한다.

#### 2.1. 기본 서버 설정

* 64비트 버전의 Java 17이 필요하며 최소 버전 17.0.3이 필요하다.
* 파이썬 버전 2.6.x, 2.7.x 또는 3.x이 필요하다.
* 파일 오픈 개수 설정
```shell
[igkim@trino ~]$ vi /etc/security/limits.conf
```
```
igkim     soft    nofile  131072
igkim     hard    nofile  131072
```

#### 2.2. 설치 및 환결 설정

* 다운로드 및 압축 해제(압축 푼 디렉토리를 $TRINO_HOME 으로 정의한다.)  
https://repo1.maven.org/maven2/io/trino/trino-server/392/trino-server-392.tar.gz

* 노드 설정
```
[igkim@trino ~]$ vi $TRINO_HOME/etc/node.properties
```
```
# Cluster name (소문자 영숫자 문자로 시작해야 하며 소문자 영숫자 또는 밑줄(_) 문자만 포함할 수 있다.)
node.environment=production

# 해당 node name (영숫자 문자로 시작해야 하며 영숫자, - 또는 _ 문자만 포함할 수 있다.)
node.id=trino-coordinator

# Log 및 기타 데이터 저장 경로
node.data-dir=/var/trino/data
```

* JVM 설정
```
[igkim@trino ~]$ vi $TRINO_HOME/etc/jvm.config
```
```
-server
-Xmx16G
-XX:InitialRAMPercentage=80
-XX:MaxRAMPercentage=80
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-XX:ReservedCodeCacheSize=512M
-XX:PerMethodRecompilationCutoff=10000
-XX:PerBytecodeRecompilationCutoff=10000
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
-XX:+UnlockDiagnosticVMOptions
-XX:+UseAESCTRIntrinsics
```










