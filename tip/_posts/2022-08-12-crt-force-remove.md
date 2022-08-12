---
layout: post
author: author1
title: HTTPS 인증서 강제 삭제하기
description: >

---

# HTTPS 인증서 강제 삭제하기

브라우저에서 인증서를 삭제하기 위해서는
크롬 기준, 설정 - 개인정보 및 보안 - 보안 - 인증서 관리 메뉴로 들어간다.

그러나 다음과 같이 인증서를 제거할 수 없는 경우가 있다.

![crt-01](/img/crt-01.png)

이 때는 Window + r 단축키를 사용하여 실행 창을 열고 `certmgr.msc` 를 입력하여 인증서 관리자를 실행한다.

![crt-02](/img/crt-02.png)

다음과 같이 삭제할 인증서 선택 후 삭제를 누르면 끝.