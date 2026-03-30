---
title: "예시 세미나 - Astro로 정적 사이트 만들기"
date: "2026-03-30"
category: "웹 개발"
tags: ["Astro", "SSG", "Cloudflare"]
description: "Astro 프레임워크를 활용한 정적 사이트 구축 및 Cloudflare Pages 배포 방법을 다룹니다."
---

## 개요

이 세미나에서는 Astro 프레임워크의 핵심 개념과 Cloudflare Pages를 활용한 배포 방법을 소개합니다.

## 목차

1. Astro란?
2. Content Collections
3. Cloudflare Pages 배포

## Astro란?

Astro는 콘텐츠 중심 웹사이트에 최적화된 정적 사이트 생성기입니다.

- **Zero JS by default** - 클라이언트에 불필요한 JavaScript를 보내지 않음
- **Content Collections** - 마크다운/MDX 기반의 타입 안전한 콘텐츠 관리
- **Island Architecture** - 필요한 부분만 하이드레이션

## 새 세미나 추가 방법

`src/content/seminars/` 디렉토리에 마크다운 파일을 추가하면 됩니다:

```markdown
---
title: "세미나 제목"
date: "2026-04-01"
category: "카테고리"
tags: ["태그1", "태그2"]
description: "한줄 설명"
---

본문 내용...
```
