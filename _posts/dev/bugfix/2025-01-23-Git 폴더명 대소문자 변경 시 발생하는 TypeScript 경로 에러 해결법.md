---
layout: post
title: Git 폴더명 대소문자 변경 시 발생하는 TypeScript 경로 에러 해결법
description: >
   Git에서 폴더명을 대문자에서 소문자로 변경할 때 발생하는 import 경로 에러와 완전한 해결 방법
sitemap: true
image: /assets/img/sidebar-bg0.jpg
hide_last_modified: false
tags: [Git, TypeScript, Next.js, macOS, 버그 해결, 폴더명, case-sensitive]
keywords: Git, TypeScript, Next.js, macOS, 버그 해결, 폴더명, case-sensitive, import 에러
categories: [dev, bugfix]
---

# <span style="color: #1976D2;">Git 폴더명 대소문자 변경 시 발생하는 TypeScript 경로 에러 해결법</span>

프론트엔드 개발 중 폴더명을 대문자에서 소문자로 변경했는데 갑자기 모든 import 경로에서 "모듈을 찾을 수 없습니다" 에러가 발생한 경험이 있으신가요? 이는 Git과 macOS 파일시스템의 특성으로 인해 발생하는 흔한 문제입니다. 오늘은 이 문제의 원인과 완전한 해결 방법에 대해 알아보겠습니다.

> <small style="background-color: #FFEBEE; padding: 2px 5px; border-radius: 3px; color: #C62828;">⚠️ 주의: 단순한 git mv 명령어만으로는 해결되지 않습니다.</small>

## <span style="color: #1976D2;">목차</span>

1. [문제 상황](#1-문제-상황)
2. [근본 원인 분석](#2-근본-원인-분석)
3. [해결 과정](#3-해결-과정)
4. [검증 방법](#4-검증-방법)
5. [예방책](#5-예방책)

---

## <span style="color: #1976D2;">1. 문제 상황</span>

### 발생한 증상

React/Next.js 프로젝트에서 다음과 같은 상황이 발생했습니다:

```
# 변경 전 폴더 구조
src/pages/portfolio-legacy/ui/Aside/
src/pages/portfolio-legacy/ui/Header/

# 변경 후 폴더 구조
src/pages/portfolio-legacy/ui/aside/
src/pages/portfolio-legacy/ui/header/
```

폴더명을 대문자에서 소문자로 변경한 후 다음과 같은 에러들이 발생했습니다:

- **TypeScript 컴파일 에러**: "Cannot find module" 에러
- **절대 경로 import 실패**: `@/pages/portfolio-legacy/ui/aside` 경로를 찾을 수 없음
- **개발 서버 실행 불가**: Next.js dev 서버가 모듈을 찾지 못함

### 에러 메시지 예시

```bash
Module not found: Can't resolve '@/pages/portfolio-legacy/ui/aside'
TS2307: Cannot find module '@/pages/portfolio-legacy/ui/aside/Navigation' 
or its corresponding type declarations.
```

---

## <span style="color: #1976D2;">2. 근본 원인 분석</span>

이 문제가 발생하는 근본적인 이유는 세 가지 시스템의 특성이 결합되어 나타나는 현상입니다.

### 2.1 Git의 case-insensitive 특성

Git은 기본적으로 **대소문자를 구분하지 않습니다(case-insensitive)**. 이는 다음을 의미합니다:

```bash
# Git 설정 확인
git config core.ignorecase
# true (기본값)
```

- Git 히스토리에는 여전히 `Aside`, `Header`로 기록됨
- 폴더명 변경이 Git에 의해 인식되지 않음
- `git status`에서 변경사항이 감지되지 않음

### 2.2 macOS 파일시스템의 특성

macOS의 **APFS(Apple File System)**도 기본적으로 case-insensitive입니다:

```bash
# 파일시스템 확인
diskutil info / | grep "Case-Sensitive"
# Case-Sensitive:              No
```

- 실제 파일시스템에서는 소문자 폴더가 생성됨
- 하지만 Git은 이 변경을 인식하지 못함
- 결과적으로 Git 히스토리와 실제 파일시스템 간 불일치 발생

### 2.3 TypeScript/Next.js 캐시 시스템

TypeScript와 Next.js는 모듈 해석을 위해 캐시를 사용합니다:

- **TypeScript Language Server**: 모듈 경로를 메모리에 캐시
- **Next.js 빌드 캐시**: `.next/` 폴더에 컴파일된 결과 저장
- **Node.js 모듈 캐시**: require/import 경로를 캐시

이러한 캐시들이 이전 대문자 경로를 계속 참조하면서 에러가 발생합니다.

---

## <span style="color: #1976D2;">3. 해결 과정</span>

다음은 실제로 문제를 해결한 단계별 과정입니다.

### 3.1 1단계: 폴더명을 원래 대문자로 복원

먼저 Git이 인식할 수 있는 상태로 되돌립니다:

```bash
# 현재 소문자 폴더를 대문자로 변경
mv src/pages/portfolio-legacy/ui/aside src/pages/portfolio-legacy/ui/Aside
mv src/pages/portfolio-legacy/ui/header src/pages/portfolio-legacy/ui/Header
```

이 단계가 필요한 이유:
- Git이 기존에 알고 있는 폴더명으로 되돌려야 함
- 이후 Git 명령어가 정상적으로 작동할 수 있음

### 3.2 2단계: Git mv로 올바른 폴더명 변경

**임시 폴더명을 거쳐서** 최종 소문자로 변경합니다:

```bash
# Aside 폴더 변경
git mv src/pages/portfolio-legacy/ui/Aside src/pages/portfolio-legacy/ui/aside-temp
git mv src/pages/portfolio-legacy/ui/aside-temp src/pages/portfolio-legacy/ui/aside

# Header 폴더 변경
git mv src/pages/portfolio-legacy/ui/Header src/pages/portfolio-legacy/ui/header-temp
git mv src/pages/portfolio-legacy/ui/header-temp src/pages/portfolio-legacy/ui/header
```

**왜 임시 폴더명을 거쳐야 할까요?**
- Git은 `Aside → aside` 직접 변경을 감지하지 못함
- 임시 폴더명(`aside-temp`)을 거치면 Git이 변경사항을 명확히 인식
- 2단계 변경으로 대소문자 변경이 Git 히스토리에 정확히 기록됨

### 3.3 3단계: 파일 내용 안전하게 이관

폴더 구조 변경 후 내부 파일들을 안전하게 처리합니다:

```bash
# 기존 파일들을 임시 이름으로 복사
cp Navigation.tsx Navigation_backup.tsx
cp index.ts index_backup.ts

# 원본 파일들 삭제
rm Navigation.tsx index.ts

# Git에 변경사항 커밋
git add .
git commit -m "feat: 폴더명 대소문자 변경 - 파일 정리"

# 백업 파일들을 원래 이름으로 복원
mv Navigation_backup.tsx Navigation.tsx
mv index_backup.ts index.ts
```

이 과정이 필요한 이유:
- 파일 내용 손실 방지
- Git 히스토리에 모든 변경사항 기록
- 단계별 작업으로 문제 발생 시 롤백 가능

### 3.4 4단계: 전체 캐시 초기화

모든 관련 캐시를 완전히 초기화합니다:

```bash
# Git 전체 캐시 지우기
git rm -r --cached .
git add .
git commit -m "fix: Git 캐시 초기화로 폴더명 변경 반영"

# Next.js 캐시 삭제
rm -rf .next
rm -rf node_modules/.cache

# TypeScript 캐시 삭제 (있는 경우)
rm -rf .tsbuildinfo
```

각 캐시 삭제의 역할:
- `git rm --cached`: Git 인덱스에서 모든 파일 제거 후 재추가
- `.next`: Next.js 빌드 캐시 완전 삭제
- `node_modules/.cache`: 번들러 캐시 삭제

### 3.5 5단계: 개발 환경 재시작

모든 개발 도구를 재시작합니다:

```bash
# VS Code에서 TypeScript 서버 재시작
# Cmd + Shift + P → "TypeScript: Restart TS Server"

# VS Code 완전 재시작
# VS Code 종료 → 터미널에서 프로젝트 폴더 재실행
code .
```

추가 재시작 항목:
- **터미널 세션**: 새로운 터미널 창 열기
- **개발 서버**: `yarn dev` 재실행
- **ESLint 서버**: VS Code에서 자동 재시작됨

### 3.6 6단계: import 경로 확인 및 수정

모든 import 경로가 새로운 소문자 폴더명을 올바르게 참조하는지 확인:

```typescript
// 수정 전 (에러 발생)
import { Navigation } from '@/pages/portfolio-legacy/ui/Aside/Navigation';

// 수정 후 (정상 동작)
import { Navigation } from '@/pages/portfolio-legacy/ui/aside/Navigation';
```

**자동 확인 방법:**
```bash
# 모든 import 경로 검색
grep -r "portfolio-legacy/ui/[A-Z]" src/
```

---

## <span style="color: #1976D2;">4. 검증 방법</span>

해결이 완료되었는지 다음 방법들로 확인할 수 있습니다:

### 4.1 TypeScript 컴파일 확인

```bash
# TypeScript 컴파일 에러 체크
npx tsc --noEmit

# 성공 시 출력 없음
# 실패 시 구체적인 에러 메시지 표시
```

### 4.2 개발 서버 정상 실행

```bash
# Next.js 개발 서버 실행
yarn dev

# 성공 시: "ready - started server on ..."
# 실패 시: 모듈 에러 메시지 출력
```

### 4.3 Git 상태 확인

```bash
# Git 상태에서 폴더명이 소문자로 표시되는지 확인
git ls-files | grep "portfolio-legacy/ui"

# 예상 출력:
# src/pages/portfolio-legacy/ui/aside/Navigation.tsx
# src/pages/portfolio-legacy/ui/header/index.ts
```

### 4.4 절대 경로 import 테스트

VS Code에서 자동완성을 통해 경로가 올바르게 인식되는지 확인:

```typescript
// VS Code에서 자동완성 테스트
import { } from '@/pages/portfolio-legacy/ui/aside/
//                                            ↑ 
// 자동완성에서 Navigation이 표시되어야 함
```

---

## <span style="color: #1976D2;">5. 예방책</span>

향후 동일한 문제를 방지하기 위한 방법들입니다:

### 5.1 프로젝트 초기 설정

프로젝트 시작 시 폴더명 컨벤션을 명확히 정의:

```javascript
// .eslintrc.js에 폴더명 규칙 추가 (예시)
module.exports = {
  rules: {
    // 폴더명은 kebab-case 사용
    'unicorn/filename-case': [
      'error',
      {
        cases: {
          kebabCase: true,
        },
      },
    ],
  },
};
```

### 5.2 Git 설정 변경

대소문자 구분을 활성화하여 문제 자체를 방지:

```bash
# 현재 저장소에서만 적용
git config core.ignorecase false

# 전역 설정 (새로운 저장소에 기본 적용)
git config --global core.ignorecase false
```

**주의사항**: 이 설정은 팀 전체가 동일하게 적용해야 하므로 팀 협의 필요

### 5.3 폴더명 변경 시 표준 절차

폴더명 변경 시 반드시 Git 명령어 사용:

```bash
# ❌ 잘못된 방법
mv src/components/Button src/components/button

# ✅ 올바른 방법
git mv src/components/Button src/components/button-temp
git mv src/components/button-temp src/components/button
```

### 5.4 개발 환경 체크리스트

팀에서 공유할 수 있는 체크리스트:

```markdown
## 폴더명 변경 시 체크리스트

- [ ] Git mv 명령어 사용 (임시 폴더명 경유)
- [ ] 모든 import 경로 수정
- [ ] Git 캐시 초기화 (git rm --cached .)
- [ ] Next.js 캐시 삭제 (rm -rf .next)
- [ ] TypeScript 서버 재시작
- [ ] 개발 서버 정상 실행 확인
- [ ] 팀원들에게 변경사항 공지
```

---

## <span style="color: #1976D2;">마무리</span>

Git에서 폴더명 대소문자 변경 시 발생하는 문제는 Git, macOS, TypeScript/Next.js의 특성이 결합되어 나타나는 복합적인 이슈입니다. 단순한 방법으로는 해결되지 않지만, 체계적인 접근을 통해 완전히 해결할 수 있습니다.

핵심은 **Git이 변경사항을 명확히 인식할 수 있도록** 임시 폴더명을 거쳐 변경하고, **모든 관련 캐시를 초기화**하는 것입니다. 또한 **사전 예방**을 통해 이런 문제 자체가 발생하지 않도록 하는 것이 가장 좋은 해결책입니다.

이 가이드가 비슷한 문제를 겪고 있는 개발자들에게 도움이 되기를 바라며, 더 나은 개발 경험을 위해 사전 예방책도 함께 적용해보시기 바랍니다.

> <small style="background-color: #E8F5E8; padding: 2px 5px; border-radius: 3px; color: #2E7D32;">✅ 성공적으로 해결되면 모든 import 경로가 정상 작동하고 개발 서버가 문제없이 실행됩니다.</small>
