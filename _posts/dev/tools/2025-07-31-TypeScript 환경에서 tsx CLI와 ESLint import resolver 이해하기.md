---
layout: post
title: "TypeScript 환경에서 tsx CLI와 ESLint import/resolver 이해하기"
subtitle: "경로 별칭 해석 문제를 해결하는 두 가지 접근법"
category: dev
tags: [tools, typescript, tsx, eslint, module-resolution]
image:
  path: /assets/img/blog/tsx-eslint-resolver.png
---

TypeScript 프로젝트를 개발하다 보면 경로 별칭(path aliases)과 모듈 해석에서 복잡한 문제들을 마주하게 됩니다. 특히 ESM 환경에서 TypeScript의 경로 별칭이 제대로 작동하지 않거나, ESLint에서 모듈을 찾지 못하는 상황이 발생하죠. 이번 글에서는 tsx CLI와 ESLint의 import/resolver 설정을 통해 이런 문제들을 어떻게 해결할 수 있는지 자세히 알아보겠습니다.

## 🚨 ESM 환경에서의 경로 별칭 문제

### 문제 상황

ESM(ECMAScript Modules)은 기본적으로 TypeScript의 경로 별칭을 인식하지 못합니다. 이는 ESM이 JavaScript 표준이고, TypeScript의 경로 별칭은 TypeScript 컴파일러의 기능이기 때문입니다.

예를 들어, 다음과 같은 TypeScript 설정이 있다고 가정해봅시다:

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@utils/*": ["./src/utils/*"]
    }
  }
}
```

```typescript
// 코드에서 이렇게 사용
import { Button } from '@components/Button';
import { formatDate } from '@utils/date';
```

TypeScript 컴파일러는 이 경로들을 올바르게 해석하지만, 런타임에서는 다음과 같은 에러가 발생할 수 있습니다:

```
Error [ERR_MODULE_NOT_FOUND]: Cannot resolve module '@components/Button'
```

### 왜 이런 문제가 발생할까요?

1. **컴파일 시점 vs 런타임**: TypeScript 컴파일러는 경로 별칭을 실제 파일 경로로 변환하지 않고, 타입 체킹만 수행합니다.
2. **ESM의 엄격한 모듈 해석**: ESM은 상대 경로나 절대 경로만 인식하며, 별칭은 이해하지 못합니다.
3. **Node.js의 모듈 해석 알고리즘**: Node.js는 TypeScript 설정을 참조하지 않습니다.

## 🛠️ tsx CLI를 통한 해결

### tsx CLI란?

tsx CLI는 esbuild를 기반으로 한 TypeScript 실행 도구입니다. Node.js의 `--loader` 훅을 사용하여 TypeScript 코드를 빠르게 변환하고 실행합니다.

### tsx CLI 설치 및 사용

```bash
# 설치
npm install --save-dev tsx

# 사용
npx tsx src/index.ts

# 또는 package.json 스크립트로
```

```json
// package.json
{
  "scripts": {
    "dev": "tsx src/index.ts",
    "dev:watch": "tsx watch src/index.ts"
  }
}
```

### tsx CLI가 경로 별칭을 해결하는 방법

tsx는 다음과 같은 방식으로 경로 별칭 문제를 해결합니다:

1. **tsconfig.json 읽기**: tsx는 프로젝트의 `tsconfig.json` 파일을 자동으로 찾아 읽습니다.
2. **경로 매핑 처리**: `paths` 설정을 인식하여 런타임에 실제 파일 경로로 변환합니다.
3. **esbuild 통합**: esbuild의 빠른 변환 속도를 활용하여 즉시 실행합니다.

### 실제 동작 예시

```typescript
// src/index.ts
import { Button } from '@components/Button';  // 별칭 사용
import { formatDate } from '@utils/date';

// tsx가 런타임에 다음과 같이 해석
// import { Button } from './src/components/Button';
// import { formatDate } from './src/utils/date';
```

## 🔍 ESLint import/resolver 설정

### ESLint에서 발생하는 문제

tsx CLI를 사용하더라도 ESLint는 여전히 경로 별칭을 인식하지 못할 수 있습니다. ESLint는 정적 분석 도구로, tsx CLI의 런타임 동작을 고려하지 않기 때문입니다.

다음과 같은 ESLint 오류가 발생할 수 있습니다:

```
Unable to resolve path to module '@components/Button'
'@components/Button' should be listed in the project's dependencies
```

### import/resolver 설정이란?

`import/resolver` 설정은 ESLint가 모듈 경로를 어떻게 해석할지 결정하는 설정입니다. 이를 통해 ESLint가 TypeScript의 경로 별칭을 올바르게 이해할 수 있도록 도울 수 있습니다.

### 필요한 패키지 설치

```bash
npm install --save-dev eslint-import-resolver-typescript
```

### 기본 설정

```json
// .eslintrc.json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true,
        "project": "./tsconfig.json"
      }
    }
  }
}
```

### 고급 설정 옵션

```json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true,
        "project": [
          "./tsconfig.json",
          "./packages/*/tsconfig.json"  // 모노레포의 경우
        ],
        "extensions": [".ts", ".tsx", ".js", ".jsx"]
      }
    }
  }
}
```

### 다중 resolver 설정

```json
{
  "settings": {
    "import/resolver": {
      "node": {
        "extensions": [".js", ".jsx", ".ts", ".tsx"]
      },
      "typescript": {
        "alwaysTryTypes": true
      }
    }
  }
}
```

## 🔄 tsx CLI와 import/resolver의 관계

### 서로 다른 시점에서의 해결

이 두 도구는 같은 문제(경로 별칭 해석)를 서로 다른 컨텍스트에서 해결합니다:

**tsx CLI**:
- **시점**: 런타임 (코드 실행 시)
- **방법**: esbuild를 통한 실시간 변환
- **목적**: 실제 코드 실행

**ESLint import/resolver**:
- **시점**: 정적 분석 (코드 작성/린팅 시)
- **방법**: TypeScript 컴파일러 API 활용
- **목적**: 코드 품질 검사

### 어떻게 두 방식이 모두 가능한가?

두 도구 모두 `tsconfig.json`의 설정을 기반으로 동작하기 때문에 일관된 경로 해석이 가능합니다:

1. **공통 기반**: 둘 다 `tsconfig.json`의 `paths` 설정을 참조
2. **다른 구현**: 
   - tsx는 esbuild의 path mapping 기능 활용
   - ESLint resolver는 TypeScript 컴파일러 API 활용
3. **동일한 결과**: 같은 설정 파일을 참조하므로 일관된 해석

## 🎯 실무에서의 설정 예시

### 완성된 프로젝트 설정

**tsconfig.json**:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler", // 또는 "node"
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@utils/*": ["./src/utils/*"],
      "@types/*": ["./src/types/*"]
    },
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  }
}
```

**package.json**:
```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx src/index.ts",
    "dev:watch": "tsx watch src/index.ts",
    "build": "tsc",
    "lint": "eslint src --ext .ts,.tsx"
  }
}
```

**.eslintrc.json**:
```json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "import"],
  "settings": {
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true,
        "project": "./tsconfig.json"
      }
    }
  },
  "rules": {
    "import/no-unresolved": "error",
    "import/named": "error",
    "import/default": "error"
  }
}
```

### Next.js 프로젝트에서의 설정

Next.js는 13버전부터 App Router를 도입했지만, 경로 별칭을 자체적으로 지원합니다. 다만 ESLint 설정은 여전히 필요할 수 있습니다:

```json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true
      },
      "node": {
        "extensions": [".js", ".jsx", ".ts", ".tsx"]
      }
    }
  }
}
```

## 🔧 문제 해결 및 디버깅

### 일반적인 문제들

1. **tsconfig.json을 찾지 못하는 경우**:
```json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "project": "./tsconfig.json"  // 명시적 경로 지정
      }
    }
  }
}
```

2. **모노레포에서의 설정**:
```json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "project": [
          "./tsconfig.json",
          "./apps/*/tsconfig.json",
          "./packages/*/tsconfig.json"
        ]
      }
    }
  }
}
```

3. **캐시 문제**:
```bash
# ESLint 캐시 삭제
rm -rf .eslintcache

# TypeScript 빌드 정보 삭제
rm -rf .tsbuildinfo
```

### 디버깅 방법

ESLint resolver가 제대로 작동하는지 확인:

```bash
# 디버그 모드로 ESLint 실행
DEBUG=eslint-import-resolver-typescript npx eslint src/index.ts
```

## 🚀 성능 최적화

### tsx CLI 최적화

```json
// package.json
{
  "scripts": {
    "dev": "tsx --conditions=development src/index.ts",
    "dev:inspect": "tsx --inspect src/index.ts"
  }
}
```

### ESLint resolver 최적화

```json
{
  "settings": {
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true,
        "project": "./tsconfig.json",
        "extensions": [".ts", ".tsx"], // 필요한 확장자만 지정
        "conditionNames": ["types", "import"] // 조건부 exports 지원
      }
    }
  }
}
```

## 📊 비교 및 선택 가이드

### 언제 tsx CLI를 사용해야 할까?

- **개발 환경에서 빠른 실행**이 필요한 경우
- **경로 별칭이 많이 사용**되는 프로젝트
- **빌드 없이 바로 실행**하고 싶은 경우
- **Hot reload**가 필요한 개발 서버

### 언제 import/resolver 설정이 중요한가?

- **팀 협업**에서 일관된 코드 품질 유지
- **CI/CD 파이프라인**에서 린팅 검사
- **IDE 통합**으로 실시간 오류 표시
- **리팩토링 시 안전성** 확보

## 마무리

tsx CLI와 ESLint의 import/resolver 설정은 각각 런타임과 정적 분석 단계에서 TypeScript의 경로 별칭 문제를 해결하는 방법입니다. 두 도구를 함께 사용하면 TypeScript 프로젝트에서 비교적 일관된 모듈 해석을 경험할 수 있습니다.

이런 설정들이 처음에는 복잡해 보일 수 있지만, 한 번 제대로 구성해 놓으면 코드 가독성을 높이고 에러를 미리 방지하는 데 도움이 됩니다. 특히 다인 프로젝트나 팀 개발에서는 이런 도구들이 더 유용할 수 있습니다.
