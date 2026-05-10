# web-security-lab

> 웹 취약점 분석 실습 — bWAPP 기반 HTML Injection · XSS 공격 시나리오 및 방어 방법

의도적으로 취약하게 설계된 웹 애플리케이션 bWAPP을 VirtualBox 환경에서 구축하고, 실제 공격 시나리오를 통해 웹 취약점의 동작 원리와 방어 방법을 학습한 실습 기록입니다.

---

## 사용 기술

- VirtualBox (bee-box 가상 머신)
- bWAPP (Buggy Web Application)
- Burp Suite (Kali Linux)
- HTML Injection / XSS / SQL Injection

---

## 실습 환경

| 구성 | 내용 |
|------|------|
| 취약 앱 | bWAPP (bee-box VM) |
| 공격 도구 | Burp Suite (Kali Linux) |
| 네트워크 | VirtualBox 브릿지 어댑터 |
| 계정 | bee / bug |

---

## 실습 항목

### HTML Injection — Reflected GET
**Low**: `<h1>H1</h1>` 입력 → 태그 그대로 렌더링 확인

**XSS — 경고창 실행**
```
<script>alert("test")</script>
```

**쿠키 탈취 시나리오**
```
<script>alert(document.cookie)</script>
```
세션 쿠키 노출 확인 — 실제 공격 시 외부 서버로 쿠키 전송해 세션 하이재킹 가능

**Medium — URL 인코딩 우회**
```
%3Ch1%3ESuccess%3C%2Fh1%3E
```

**High** — 인코딩까지 서버에서 처리해 공격 차단됨

### HTML Injection — Stored Blog
```html
<script>alert("test")</script>
```
저장 시 다른 사용자 접속 때 자동 실행 — Reflected XSS 대비 피해 범위가 훨씬 넓음

### Burp Suite 패킷 분석
- Intercept On으로 GET 요청 패킷 가로채기
- 요청/응답 헤더 구조 분석

---

## 배운 점

보안 수준(Low / Medium / High)에 따라 동일한 공격이 막히거나 통과되는 과정을 직접 보면서, 방어가 어느 레이어에서 어떻게 이루어지는지 체감했습니다.

특히 `httpOnly` 쿠키 플래그가 JavaScript에서 `document.cookie` 접근을 막아 XSS 기반 세션 탈취를 차단한다는 원리를 실습으로 확인한 것이 인상 깊었습니다. 백엔드에서 세션 쿠키를 설정할 때 `httpOnly: true` 한 줄이 왜 중요한지를 직접 눈으로 보게 됐습니다.

---

## 어려웠던 점

VirtualBox 네트워크 설정에서 공격 머신(Kali)과 대상 머신(bee-box)이 서로 통신하도록 브릿지 어댑터를 잡는 과정이 까다로웠습니다. IP 대역을 맞추는 데 시간이 많이 소요됐고, 네트워크 설정을 잘못하면 Burp Suite 프록시가 요청을 가로채지 못하는 문제가 반복됐습니다.

Burp Suite의 Intercept 기능을 처음 사용할 때 인터셉트를 켠 채로 두면 브라우저가 멈추는 것처럼 보여서 혼란스러웠는데, 요청이 중간에서 대기 중인 상태라는 것을 이해하고 나서 흐름이 잡혔습니다.

---

## 아쉬운 점 및 개선 방향

- HTML Injection과 XSS만 실습했는데, SQL Injection이나 CSRF 등 다른 OWASP Top 10 취약점도 실습해보고 싶습니다.
- 공격만 해봤고 방어 코드를 직접 작성해보지 못했습니다. Node.js나 Spring Boot 코드에서 입력값 이스케이프 처리, CSP 헤더 설정 등을 직접 구현해보는 것이 다음 목표입니다.

---

## 취약점별 방어 방법 정리

| 취약점 | 방어 |
|--------|------|
| Reflected XSS | 서버 측 입력값 이스케이프 (`htmlspecialchars`) |
| Stored XSS | DB 저장 전 sanitize, 출력 시 이스케이프 |
| 쿠키 탈취 | `httpOnly: true` 쿠키 플래그 설정 |
| 전반적 XSS | Content-Security-Policy(CSP) 헤더 설정 |

---

## 추가로 공부해야 할 내용

- OWASP Top 10 전체 취약점 개요
- SQL Injection 원리와 Prepared Statement 방어
- CSRF 공격과 CSRF Token 방어
- CSP(Content Security Policy) 헤더 설정
- 방어 코드 직접 작성 실습
