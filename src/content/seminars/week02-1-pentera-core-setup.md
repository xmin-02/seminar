---
title: "Week 02-1 — Pentera Core 플랫폼 설정 & 테스트 시나리오"
date: "2026-03-23"
category: "Pentera"
tags: ["Pentera Core", "Black Box", "Gray Box", "Targeted"]
description: "Pentera Core 초기 설정 7단계, 워크플로우, Black Box·Gray Box·Targeted 테스트 시나리오를 비교합니다."
---

## 개요

2주차 1차시에서는 Pentera Core를 처음 설치하고 설정하는 과정부터, 테스트 시나리오를 어떻게 선택하고 구성하는지까지 다룹니다.

## 학습 목표

1. Pentera Core 초기 설정 절차를 이해하고 수행
2. Black Box, Gray Box, Targeted 세 가지 테스트 시나리오의 차이를 명확히 설명
3. 각 시나리오의 파라미터를 직접 구성
4. 테스트 스케줄링과 자동화 설정 이해
5. Safety Guardrails의 작동 원리 설명

## 초기 설정 시퀀스 (7단계)

| 단계 | 내용 |
|------|------|
| Step 1 | 워크스테이션에서 Pentera 머신까지 네트워크 연결 확인 (`ping`) |
| Step 2 | Chrome 또는 Edge 브라우저 실행 (Firefox 미지원) |
| Step 3 | `https://<target_machine>:8181` 접속 |
| Step 4 | 라이선스 업로드 — 테스트 가능한 IP 범위의 최외곽 경계 정의 |
| Step 5 | 첫 번째 Admin 사용자 생성 |
| Step 6 | EULA 동의 및 Management Server 등록 |
| Step 7 | 로그인 및 2FA 설정 (Google Authenticator 등) |

> **주의**: 첫 번째 Admin 사용자는 한 번 생성하면 변경하거나 삭제할 수 없습니다.

## 초기 구성

### 필수

- **License IP Ranges** — Pentera가 테스트할 수 있는 IP의 최외곽 경계. 이 범위 밖의 IP는 어떤 시나리오에서도 절대 테스트되지 않음

### 권장

- **User Display Settings** — UI 환경설정, 비밀번호 마스크 여부
- **User Management** — Admin, Operator, View-Only 역할 부여
- **Critical Assets** — 도메인 컨트롤러, 결제 서버 등 고가치 타겟 지정 → 자동으로 9.9 Critical 플래그
- **Email Integration** — 테스트 시작/종료 시 자동 알림
- **SIEM Integration** — Splunk, QRadar 등에 실시간 전송
- **Residue Cleanup** — 테스트 후 아티팩트 자동 제거

## The Pentera Workflow (7단계)

1. **Connect** — Pentera 서버를 대상 네트워크와 동일 브로드캐스트 도메인에 배치
2. **Create Scenario** — 시나리오 이름, IP 범위, 지속시간, 허용 액션, 스케줄 정의
3. **Run Test** — 수동 실행 또는 반복 스케줄 설정
4. **Observe Live** — 실시간 결과 확인, 대기 중 액션 승인, 동적 User Input 추가
5. **Review Results** — 취약점, Achievement, Attack Map 분석 및 리포트 생성
6. **Review Vulnerabilities** — 네트워크 컨텍스트에 맞는 우선순위별 조치 권고
7. **Remediate & Validate** — 취약점 수정 후 재테스트하여 조치 효과 확인

> Step 7이 Pentera의 핵심 가치입니다. 다른 도구들은 "발견"에서 끝나지만, Pentera는 **"수정이 정말 작동하는가?"**까지 검증합니다.

## 테스트 시나리오 유형

### Black Box — "The Outsider"

외부 공격자를 시뮬레이션합니다. 사전 정보 없이, 자격증명 없이 시작합니다.

- **목표**: 진입점, 약한 비밀번호, 잘못된 구성 발견
- **용도**: 기준 보안 평가

**Attack Vectors 설정**:
- Network Discovery (ARP/Ping) — ENABLED
- Vulnerability Scanning — ENABLED
- Sniffing (LLMNR/NBT-NS) — ENABLED
- Relay Attacks (SMB/HTTP) — ENABLED
- Exploitation (RCE) — ENABLED
- Credential Injection — **DISABLED** (자격증명 없음)

### Gray Box — "The Insider"

침해된 호스트나 악의적 내부자를 시뮬레이션합니다.

- **목표**: 수평 이동, 권한 상승, 핵심 데이터 접근
- **용도**: 내부 분리와 방어 깊이 테스트

**세 가지 "What-If" 시나리오**:
1. **Malicious Insider / Rogue Device** — 네트워크 포트에 노트북을 연결한 공격자
2. **Phishing Victim** — 피싱으로 감염된 직원 워크스테이션
3. **Credential Leak** — 다크웹에서 유효한 AD 자격증명을 구매한 공격자

### Targeted — "The Sniper"

특정 취약점이나 공격 벡터 검증에 집중합니다.

- **목표**: 패치 검증, 위협 헌팅
- **시작점**: Log4j, PrintNightmare 같은 특정 CVE

**CISA KEV Integration**: 실제 야생에서 활발히 사용되는 취약점 카탈로그와 직접 통합

| CVE | 이름 | 상태 |
|-----|------|------|
| CVE-2023-23397 | Outlook | ACTIVE |
| CVE-2021-44228 | Log4j | ACTIVE |
| CVE-2021-34527 | PrintNightmare | ACTIVE |
| CVE-2020-1472 | Zerologon | ACTIVE |

## 고급 공격 설정

- **Kerberoasting** — SPN용 서비스 티켓 요청 → 오프라인 크래킹
- **AS-REP Roasting** — "Kerberos 사전 인증 불필요" 계정 타겟
- **SMB / NTLM Relay** — 인증 트래픽 가로채 다른 시스템으로 중계

> 고급 설정은 Purple Team 연습이나 탐지 통제 효과 테스트 시에만 활성화를 권장합니다.

## 핵심 정리

- **SETUP**: 초기 설정 7단계와 필수/권장 구성
- **WORKFLOW**: Connect에서 Remediate & Validate까지 7단계
- **SCENARIOS**: Black Box(외부), Gray Box(내부), Targeted(정밀 검증)
- **CISA KEV**: CVSS 점수만이 아닌, 실제 익스플로잇되는 취약점 기반 우선순위
