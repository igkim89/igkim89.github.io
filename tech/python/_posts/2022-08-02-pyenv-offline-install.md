---
layout: post
author: author1
title: Pyenv offline install
categories: [python]
description: >
  Python 가상환경에 사용하는 pyenv 환경 구축
---

# Pyenv offline install  

## 1. pyenv install
**외부 네트워크를 사용하지 않는 offline 환경 (e.g. private network) 에서 pyenv를 설치한다.**  
**사전에 Git, 파이썬 인터프리터 컴파일을 위한 라이브러리와 패키지 등이 필요하다.**

#### 1.1. online 환경에서 pyenv-installer 다운로드 

[https://github.com/pyenv/pyenv-installer](https://github.com/pyenv/pyenv-installer)  
[https://github.com/pyenv/pyenv-installer/archive/refs/heads/master.zip](https://github.com/pyenv/pyenv-installer/archive/refs/heads/master.zip)

```bash
[igkim@igkim-vm installer]$ wget https://github.com/pyenv/pyenv-installer/archive/refs/heads/master.zip
```

#### 1.2. pyenv package 다운로드

```bash
[igkim@igkim-vm installer]$ unzip master.zip
[igkim@igkim-vm installer]$ cd pyenv-installer-master
[igkim@igkim-vm pyenv-installer-master]$ bin/download-pyenv-package.sh
```

완료 후, bin 디렉토리에 `pyenv-package.tar.gz` 파일이 정상적으로 생성되었는지 확인한다.  
아래와 같은 에러가 발생한다면 `1.2.1. https 설정 추가` 를 수행한다.

~~~shell
fatal: unable to connect to github.com:
github.com[0: 52.78.231.108]: errno=Connection timed out
~~~

~~~shell
fatal: Unable to look up github.com (port 9418)
~~~

##### 1.2.1. https 설정 추가

https 설정을 위해 스크립트 파일을 수정한다.

~~~shell
[igkim@igkim-vm pyenv-installer-master]$ vi bin/download-pyenv-package.sh
~~~

12 line `USE_HTTPS=true` 추가

~~~shell
  1 #!/usr/bin/env bash
  2 
  3 checkout() {
  4   [ -d "$2" ] && (cd "$2"; git clone "$1")
  5 }
  6 
  7 if [ -z "$PYENV_PACKAGE_ARCHIVE" ]; then
  8   PYENV_PACKAGE_ARCHIVE="$(cd $(dirname "$0") && pwd)/pyenv-package.tar.gz"
  9 fi
 10 
 11 TMP_DIR=$(mktemp -d)
 12 USE_HTTPS=true
 13 
 14 if [ -n "${USE_HTTPS}" ]; then
 15   GITHUB="https://github.com"
 16 else
 17   GITHUB="git://github.com"
 18 fi
 19 
 20 # checkout to temporary directory.
 21 checkout "${GITHUB}/pyenv/pyenv.git"            "$TMP_DIR"
 22 checkout "${GITHUB}/pyenv/pyenv-doctor.git"     "$TMP_DIR"
 23 checkout "${GITHUB}/pyenv/pyenv-installer.git"  "$TMP_DIR"
 24 checkout "${GITHUB}/pyenv/pyenv-update.git"     "$TMP_DIR"
 25 checkout "${GITHUB}/pyenv/pyenv-virtualenv.git" "$TMP_DIR"
 26 checkout "${GITHUB}/pyenv/pyenv-which-ext.git"  "$TMP_DIR"
 27 
 28 # create archive.
 29 tar -zcf "$PYENV_PACKAGE_ARCHIVE" -C "$TMP_DIR" .
 30 
 31 rm -rf $TMP_DIR
~~~

수정 후 pyenv package를 다운로드한다.

#### 1.3. pyenv를 설치할 offline 서버로 설치 파일 전송

위에서 작업한 디렉토리 전체를 압축한다.  
```shell
[igkim@igkim-vm installer]$ tar cvfz pyenv.tgz pyenv-installer-master
```

offline 서버로 압축파일을 전송한다.

#### 1.4. pyenv 설치

압축 해제 후 offline installer를 실행한다.

```shell
[igkim@igkim-offline ~]$ tar xvfz pyenv.tgz
[igkim@igkim-offline ~]$ cd pyenv-installer-master
[igkim@igkim-offline pyenv-installer-master]$ bin/pyenv-offline-installer
```

설치 완료 후 환경변수를 설정한다.

```shell
[igkim@igkim-offline pyenv-installer-master]$ vi ~/.bashrc
```

~~~shell
export PYENV_ROOT="$HOME/.pyenv" 
export PATH="$PYENV_ROOT/bin:$PATH" 
eval "$(pyenv init --path)" 

eval "$(pyenv init -)" 
eval "$(pyenv virtualenv-init -)" 
~~~

```shell
[igkim@igkim-offline pyenv-installer-master]$ source ~/.bashrc
```

## 2. python 설치
**외부 네트워크를 사용하지 않는 offline 환경 (e.g. private network) 에서 python을 설치한다.**  
**사전에 pyenv 설치가 필요하다.**
**예제에서는 3.8.9 버전을 설치한다.**

#### 2.1. python 설치파일 다운로드

원하는 버전의 Python 설치 파일을 다운로드한다.  
[https://www.python.org/ftp/python](https://www.python.org/ftp/python)

```shell
[igkim@igkim-offline ~]$ mkdir ~/.pyenv/cache
[igkim@igkim-offline ~]$ mv Python-3.8.9.tar.xz ~/.pyenv/cache/
```

#### 2.2. python 설치

```shell
[igkim@igkim-offline ~]$ pyenv install -v 3.8.9
```

필요한 라이브러리 및 컴파일러가 설치되지 않은 경우 에러가 발생한다. 
(e.g. patch, C compiler)
추가로 필요한 라이브러리 및 컴파일러를 설치한 후 python을 다시 설치한다.

```shell
[igkim@igkim-offline ~]$ yum install patch bzip2-devel ncurses-devel readline-devel sqlite-devel openssl-devel libffi-devel
[igkim@igkim-offline ~]$ yum groupinstall "Development Tools"
```

## 3. 설치 확인

local python version 설정 후 가상환경을 생성한다.

```shell
[igkim@igkim-offline ~]$ pyenv local 3.8.9
[igkim@igkim-offline ~]$ python --version

Python 3.8.9
```

```shell
[igkim@igkim-offline ~]$ pyenv virtualenv igkim-3.8.9

(igkim-3.8.9) [igkim@igkim-docker ~]$ 
```