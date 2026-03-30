---
title: "Week 02-2 — Pentera Core 실행 모듈 & 안전 장치"
date: "2026-03-26"
category: "Pentera"
tags: ["랜섬웨어", "Password Cracking", "Credential Exposure", "Safety"]
description: "랜섬웨어 에뮬레이션, 라이브 테스트 상호작용, 패스워드 크래킹 엔진, Credential Exposure 모듈을 다룹니다."
---

## 개요

2주차 2차시에서는 실제 실행 단계로 들어갑니다. Pentera의 주요 모듈들 — Ransomware Emulation, Web Attack Surface, Password Cracking — 을 살펴보고, 테스트 실행 중 화면에서 무엇을 보고 어떤 결정을 해야 하는지, 그리고 Safety Guardrails가 어떻게 동작하는지를 학습합니다.

## 랜섬웨어 에뮬레이션 (RansomwareReady™)

별도 추가 모듈로, 실제 랜섬웨어 그룹의 공격 패턴을 에뮬레이션합니다.

**지원 캠페인**: LockBit 3.0, Conti, Maze, REvil, Cl0p, BlackCat/ALPHV, WannaCry

### 목적

EDR/AV가 랜섬웨어의 **행동**을 탐지하고 차단할 수 있는지 검증합니다. 단순 Hash Signature가 아니라 대량 파일 암호화, Shadow Copy 삭제 같은 **행동 패턴**을 테스트합니다.

### Safe Emulation

- 실제 파일을 암호화하지 않음
- Dummy File 생성 또는 Reversible Key 사용
- Production 파일 데이터 손실 **zero**

> **필수 안전 통제**: 랜섬웨어 에뮬레이션은 절대 자동으로 실행되지 않습니다. 실제 실행 시 반드시 Operator가 호스트별로 명시적으로 승인해야 합니다.

## 스케줄링 & 알림

### 자동화 테스트 사이클

| 주기 | 용도 |
|------|------|
| **Daily** | 매일 밤 경량 Vulnerability Scan → 새 노출 즉시 탐지 |
| **Weekly** | 주말 유지보수 기간에 전체 Black Box 테스트 |
| **Monthly** | 모든 모듈 포함 종합 검증 → 감사 증적 확보 |

### 알림 채널

- **Email Alerts** — SMTP를 통해 보안팀에 직접 알림, Executive Summary PDF 자동 첨부
- **Syslog / SIEM** — Splunk, QRadar 같은 SOC Dashboard와 실시간 통합

## Safety by Design (6가지 안전 장치)

> "모든 Pentera 활동은 Safe by Design. 목표는 악용 가능성의 입증이지, 피해를 주는 것이 아닙니다."

1. **Scope Enforcement** — 정의된 IP Range로 엄격히 제한
2. **Manual Approvals** — RCE, Ransomware 같은 민감한 액션은 명시적 승인 필요
3. **Stealth Levels** — 최소 영향 Stealth Scan부터 Noisy Test까지 조정 가능
4. **Sanitized Exploits** — 유해 코드나 DDoS를 실행하지 않음
5. **Full Cleanup** — 테스트 중 생성된 모든 잔여물 자동 제거
6. **Immediate Stop** — 언제든 단일 클릭으로 중단 가능

## Web & Identity 확장

### Web Attack Surface

Black-Box 웹 애플리케이션 테스팅:

- **Dynamic Crawling** — 애플리케이션 구조 자동 매핑
- **Vulnerability Scanning** — SQL Injection, XSS, LFI/RFI 등 테스트 (OWASP Top 10 커버)
- **Safe Exploitation** — `sleep` 명령 같은 무해한 Payload로 검증

### AD Password Assessment (ADPA)

"Silent Credential Auditing":

- **Zero Lockout Risk** — Domain Controller에 Brute Force 없음
- **Hash Comparison** — Read-Only 접근으로 AD Hash를 약한 비밀번호 Dictionary와 비교
- **Policy Validation** — "Password Never Expires" 같은 위험한 설정 식별

## Live Test 상호작용

테스트 실행 중 화면에서 확인할 수 있는 것들:

### Activity Log (실시간)

- Host Discovery 완료 — 24개 활성
- Port Scanning 진행 중
- MS17-010 Vulnerability 탐지 (EternalBlue)
- SMB Relay 공격 시작
- **Pending Approval** — Payload 주입 대기 중

### Dynamic User Inputs

테스트 중간에 Credential이나 Target Host를 추가할 수 있습니다. "만약 공격자가 이 정보를 얻었다면?"을 동적으로 테스트합니다.

### Pending Approvals

High Impact 액션에 대해 **APPROVE** 또는 **SKIP**을 선택합니다.

> **Human-in-the-Loop** — Automation의 효율성과 인간의 판단을 결합하는 것이 핵심입니다.

## Password Cracking Engine (4단계)

| Level | 난이도 | 방법 | 예시 |
|-------|--------|------|------|
| Level 1 | TRIVIAL | Dictionary Attack만 | `password`, `123456` |
| Level 2 | EASY | Dictionary + Mask Transformation | `password1`, `12passWORD` |
| Level 3 | MEDIUM | Dictionary + Mask + Rules | `P@ssw0rd`, `pa$$W07D123` |
| Level 4 | STRONG | 모든 방법 + Advanced Rules | `pPp@$w0orrrd123` |

> `P@ssw0rd`는 대소문자, 숫자, 특수문자를 모두 포함하지만 왜 "Medium"일까요? `Password`라는 Dictionary Word에 예측 가능한 치환 규칙(a→@, o→0)을 적용한 것이기 때문입니다. **형식적 복잡성 ≠ 보안 강도**

## Credential Exposure 모듈

Darknet Threat Intelligence를 활용하여 유출된 자격증명을 탐지합니다.

### Direct Exposure Checks

- 유효한 Username-Password 쌍의 무단 접근 노출 확인
- AD 비밀번호 Cleartext 유출 확인
- 유출된 Hash의 Cracking Resilience 테스트

### Secondary Exposure Checks

**Levenshtein Distance**로 유출 비밀번호와 현재 AD 비밀번호를 비교합니다:

```
if (Levenshtein_Distance(leaked, current) >= 0.8) {
    flag_vulnerability("High Risk Reuse");
}
```

예: 유출된 `Company2023!` → 현재 `Company2024!` = **Close Match 플래그**

## 2주차 핵심 정리

- **SETUP MASTERY**: Initial Setup 7단계와 필수/권장 구성
- **SCENARIO SELECTION**: Black Box, Gray Box, Targeted 시나리오의 목적과 Use Case
- **SAFETY FIRST**: 6가지 Safety Guardrails가 Production 환경 안전 보장
- **CONTINUOUS VALIDATION**: Scheduling과 Notification으로 지속적 보안 검증 자동화
