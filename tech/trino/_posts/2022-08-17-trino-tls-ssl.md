---
layout: post
author: author1
title: Trino TLS/SSL 적용하기
categories: [trino]
description: >
  
---

# Trino TLS/SSL 적용하기

## 1. Trino Security

기본 설정으로 Trino를 설치해보면 로그인 과정은 있지만 id는 아무렇게나 입력해도 되며 password는 입력받지 않는다.  
그러므로 보안 설정을 적용해보자.  
여러 방법이 있지만 간단한 방법 중 하나인 `Password file authentication`으로 진행을 해본다.  

하지만 `Password file authentication`을 적용하기 위해선 두 가지 선행 작업이 필요하다.  
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

* ROOT CA RSA Key 생성 (rootca.key)

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

~~~shell
[ req ]
default_bits            = 2048
default_md              = sha1
default_keyfile         = rootca.key
distinguished_name      = req_distinguished_name
extensions             = v3_ca
req_extensions = v3_ca

[ v3_ca ]
basicConstraints       = critical, CA:TRUE, pathlen:0
subjectKeyIdentifier   = hash
##authorityKeyIdentifier = keyid:always, issuer:always
keyUsage               = keyCertSign, cRLSign
nsCertType             = sslCA, emailCA, objCA
[req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = KR
countryName_min                 = 2
countryName_max                 = 2

# 회사명 입력
organizationName              = Organization Name (eg, company)
organizationName_default      = igkim

# 부서 입력
#organizationalUnitName          = Organizational Unit Name (eg, section)
#organizationalUnitName_default  = bigdata

# SSL 서비스할 domain 명 입력
commonName                      = Common Name (eg, your name or your server's hostname)
commonName_default             = trino
commonName_max                  = 64
~~~

* ROOT CSR 생성 (rootca.crt)

~~~shell
[igkim@trino rootca]$ openssl x509 -req -days 3650 -extensions v3_ca -set_serial 1 -in rootca.csr -signkey rootca.key -out rootca.crt -extfile rootca_openssl.conf

Signature ok
subject=C = KR, O = encore, CN = storage01
Getting Private key
Enter pass phrase for rootca.key: {Root RSA Key Password 입력}
~~~

#### 2.2. Trino 인증서 생성

* Trino RSA Key 생성 (trino.key)

~~~shell
[igkim@trino ~]$ rootca]$ cd ../trino
[igkim@trino trino]$ openssl genrsa -aes256 -out trino.key 2048

Generating RSA private key, 2048 bit long modulus (2 primes)
................................................+++++
......+++++
e is 65537 (0x010001)
Enter pass phrase for trino.key: {Trino RSA Key Password 입력}
Verifying - Enter pass phrase for trino.key: {Trino RSA Key Password 입력}
~~~

* Trino RSA Key Pass phrase 제거  
매번 Pass phrase를 입력해야 하는 번거로움을 방지하기 위해 제거한다.

~~~shell
[igkim@trino trino]$ cp trino.key trino.key.tmp
[igkim@trino trino]$ openssl rsa -in trino.key.tmp -out trino.key

Enter pass phrase for trino.key.tmp: {Trino RSA Key Password 입력}
writing RSA key

[igkim@trino trino]$ rm -rf trino.key.tmp
~~~

* Trino CSR Config 작성

~~~shell
[igkim@trino trino]$ vim trino_openssl.conf 
~~~

~~~shell
[ req ]
default_bits            = 2048
default_md              = sha1
default_keyfile         = ../rootca/rootca.key
distinguished_name      = req_distinguished_name
extensions             = v3_user
## req_extensions = v3_user

[ v3_user ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
authorityKeyIdentifier = keyid,issuer
subjectKeyIdentifier = hash
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth,clientAuth
subjectAltName          = @alt_names
[ alt_names]
DNS.1   = trino.bigdata.igkim
DNS.2   = trino
DNS.3   = *.bigdata.igkim

[req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = KR
countryName_min                 = 2
countryName_max                 = 2

# 회사명 입력
organizationName              = Organization Name (eg, company)
organizationName_default      = igkim

# 부서 입력
organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = bigdata

# SSL 서비스할 domain 명 입력
commonName                      = Common Name (eg, your name or your server's hostname)
commonName_default             = trino.bigdata.igkim
commonName_max                  = 64
~~~

* Trino CSR 생성 (trino.csr)

~~~shell
[igkim@trino trino]$ openssl req -new -key trino.key -out trino.csr -config trino_openssl.conf

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [KR]:
Organization Name (eg, company) [igkim]:
Organizational Unit Name (eg, section) [bigdata]:
Common Name (eg, your name or your servers hostname) [trino.bigdata.igkim]:
~~~

* Trino 인증서 발급 (trino.crt)

~~~shell
[igkim@trino trino]$ openssl x509 -req -days 365 -extensions v3_user -in trino.csr -CA ../rootca/rootca.crt -CAcreateserial -CAkey ../rootca/rootca.key -out trino.crt -extfile trino_openssl.conf 

Signature ok
subject=C = KR, O = igkim, OU = bigdata, CN = trino.bigdata.igkim
Getting CA Private Key
Enter pass phrase for ../rootca/rootca.key: {Root RSA Key Password 입력}
~~~

* Trino Keystore 생성 (trino.pem)

~~~shell
[igkim@trino trino]$ openssl pkcs12 -export -in trino.crt -inkey trino.key -out trino.pem

Enter Export Password: {Keystore Password 입력}
Verifying - Enter Export Password: {Keystore Password 입력}
~~~

#### 2.3. Trino TLS 설정

발급한 인증서를 사용하여 Trino에 TLS/SSL을 적용한다.

* 코디네이터 설정 파일 수정

~~~shell
[igkim@trino ~]$ vi $TRINO_HOME/etc/config.properties
~~~

~~~shell
coordinator=true
node-scheduler.include-coordinator=false
#http-server.http.port=18080
discovery.uri=https://192.168.1.10:18443

http-server.https.enabled=true
http-server.https.port=18443

# Trino Keystore 경로 및 Password
http-server.https.keystore.path=/home/igkim/trino/trino.pem
http-server.https.keystore.key=1313
~~~

* 서버 재시작

~~~shell
[igkim@trino ~]$ $TRINO_HOME/bin/launcher restart
~~~

#### 2.4. 인증서 배포

클라이언트 browser에서 Trino Web UI에 접속할 때 사용할 인증서를 등록해본다.

* 인증서 배포  
rootca.crt 인증서 파일을 클라이언트로 가져온다.

* 인증서 등록
  * root.ca.crt 파일을 실행한다.  
![trino-tls-01](/img/tech/trino/trino-tls-01.png)
![trino-tls-02](/img/tech/trino/trino-tls-02.png)
![trino-tls-03](/img/tech/trino/trino-tls-03.png)
![trino-tls-04](/img/tech/trino/trino-tls-04.png)

인증서 설치가 완료되었다.

## 3. Configured shared secret

클러스터에 있는 노드의 내부 인증 및 보안 통신을 사용한다.

* Secret 값 생성

~~~shell
[igkim@trino ~]$ openssl rand 512 | base64
~~~

* Trino 설정 파일 수정 (전체 노드)  
위에서 생성한 Secret 값을 `internal-communication.shared-secret` 키값에 설정한다.

~~~shell
[igkim@trino ~]$ vi $TRINO_HOME/etc/config/properties
~~~

~~~shell
internal-communication.https.required=true
internal-communication.shared-secret=Rwgcb2cdPn18euZh8BomcWQbr...
~~~

변경된 설정 값을 적용하기 위해서 서버 재시작이 필요하지만 `4. Password file authentication` 까지 진행 후 재시작한다.

## 4. Password file authentication

ID, PW를 통한 로그인 환경을 구성한다.

* Trino 설정 파일 수정 (코디네이터)

~~~shell
[igkim@trino ~]$ vi $TRINO_HOME/etc/config/properties
~~~

~~~shell
http-server.authentication.type=PASSWORD
~~~

* PW auth 설정 파일 작성

~~~shell
[igkim@trino ~]$ vi $TRINO_HOME/etc/password-authenticator.properties
~~~

~~~shell
password-authenticator.name=file
file.password-file=$TRINO_HOME/etc/password.db
~~~

* 계정 정보 파일 생성

~~~shell
[igkim@trino ~]$ touch $TRINO_HOME/etc/password.db
~~~

* 계정 추가

~~~shell
[igkim@trino ~]$ htpasswd -B -C 10 $TRINO_HOME/etc/password.db {사용할 ID}

New password: {사용할 패스워드 입력}
Re-type new password: {사용할 패스워드 입력} 
Adding password for user igkim
~~~

* 코디네이터 재시작

~~~shell
[igkim@trino ~]$ $TRINO_HOME/bin/launcher restart
~~~

## 5. Trino Web UI 접속

인증서를 배포한 클라이언트에서 Trino Web UI로 접속한다.  
![trino-tls-05](/img/tech/trino/trino-tls-05.png)

HTTPS 및 Password auth가 적용된 것을 확인할 수 있다.