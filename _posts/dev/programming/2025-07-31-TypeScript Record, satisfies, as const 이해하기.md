---
layout: post
title: "TypeScript Record, satisfies, as const 이해하기"
subtitle: "타입 안전성과 정확한 추론을 위한 현대적 TypeScript 패턴"
category: dev
tags: [programming, typescript, type-safety]
image:
  path: /assets/img/blog/typescript-patterns.png
---

TypeScript로 객체를 정의할 때 타입 안전성과 정확한 타입 추론 사이에서 고민해본 적이 있나요? 오늘은 `Record`, `satisfies`, `as const`의 조합을 통해 이 문제를 완벽하게 해결하는 방법을 알아보겠습니다.

## 🚨 기존 패턴들의 한계

먼저 TypeScript에서 객체를 정의할 때 자주 마주치는 문제들을 살펴보겠습니다.

### 패턴 1: as const만 사용했을 때의 문제

```typescript
interface EmotionCanvasEffect {
  colors: string[];
  particleCount: number;
  // ...
}

const EMOTION_CANVAS_EFFECTS = {
  joy: {
    colors: ['#f2d038', '#FFD700', '#FFA500', '#FFFFFF'],
    particleCount: 150,
    // ...
  },
  sadness: {
    colors: ['#268BD2', '#4682B4', '#87CEEB', '#B0C4DE'],
    particleCount: 80,
    // ...
  }
} as const;

// ❌ 에러 발생!
// Type 'readonly ["#f2d038", "#FFD700", "#FFA500", "#FFFFFF"]' is not assignable to type 'string[]'
```

**왜 이런 에러가 발생할까요?**

`as const`를 사용하면 배열이 `readonly ['#f2d038', '#FFD700', ...]` 타입으로 추론됩니다. 하지만 인터페이스에서는 `string[]` (mutable 배열)을 요구하므로 타입 불일치가 발생합니다.

### 패턴 2: 명시적 타입 주석만 사용했을 때의 문제

```typescript
const EMOTION_CANVAS_EFFECTS: Record<string, EmotionCanvasEffect> = {
  joy: {
    colors: ['#f2d038', '#FFD700', '#FFA500', '#FFFFFF'],
    particleCount: 150,
    // ...
  },
  sadness: {
    colors: ['#268BD2', '#4682B4', '#87CEEB', '#B0C4DE'],
    particleCount: 80,
    // ...
  }
} as const;

// ✅ 에러는 없지만...
type EmotionKeys = keyof typeof EMOTION_CANVAS_EFFECTS; 
// = string (너무 광범위!)
```

**문제점**: 타입 에러는 해결되지만, 구체적인 키 정보가 손실됩니다. `"joy" | "sadness"`가 아닌 `string`으로 추론되어 자동완성과 타입 안전성이 떨어집니다.

## 📚 Record<K, V> 타입 이해하기

### Q: Record 타입이 정확히 무엇인가요?

**A**: `Record<K, V>`는 객체의 형태를 정의하는 TypeScript 유틸리티 타입입니다.

```typescript
// Record<Keys, ValueType>의 의미
type Record<K extends keyof any, T> = {
    [P in K]: T;
}

// 실제 사용 예시
type Colors = Record<string, string>;
// 이는 다음과 같습니다:
type Colors = {
    [key: string]: string;
}

// 구체적인 키들로도 사용 가능
type RGB = Record<'red' | 'green' | 'blue', number>;
// 결과:
type RGB = {
    red: number;
    green: number;
    blue: number;
}
```

### Q: 언제 Record를 사용해야 하나요?

**A**: 다음과 같은 상황에서 유용합니다:

- 객체의 모든 값이 같은 타입일 때
- 동적인 키를 가진 객체의 타입을 정의할 때
- 인덱스 시그니처의 대안으로 사용할 때

## 🎯 satisfies 연산자의 혁신

### Q: satisfies는 어떻게 기존 문제를 해결하나요?

**A**: `satisfies`는 TypeScript 4.9에서 도입된 연산자로, "이 값이 특정 타입을 만족하는지 확인하되, 원본 타입 정보는 보존해줘"라는 의미입니다.

```typescript
// 전통적인 방법
const config: Record<string, string | number> = {
    apiUrl: "https://api.com",
    timeout: 5000,
    retries: 3
};

config.apiUrl.toUpperCase(); // ❌ 에러! string | number이므로

// satisfies 사용
const config = {
    apiUrl: "https://api.com",
    timeout: 5000,
    retries: 3
} satisfies Record<string, string | number>;

config.apiUrl.toUpperCase(); // ✅ 작동! TypeScript가 string임을 알고 있음
config.timeout * 2; // ✅ 작동! TypeScript가 number임을 알고 있음
```

### Q: 타입 주석(`:`)과 satisfies의 차이점은 무엇인가요?

**A**: 핵심 차이점은 다음과 같습니다:

- **타입 주석 (`:`)**: "이 값의 타입을 이걸로 고정해"
- **satisfies**: "이 조건을 만족하는지 체크하되, 정확한 타입은 유지해"

```typescript
// 타입 주석 사용
const person: { name: string | number } = { name: "Alice" };
person.name.toUpperCase(); // ❌ 에러! string | number에는 toUpperCase가 없음

// satisfies 사용  
const person = { name: "Alice" } satisfies { name: string | number };
person.name.toUpperCase(); // ✅ 작동! name의 실제 타입은 string이므로
```

## 🔄 패턴별 완벽 비교와 해결책

### 패턴별 동작 원리와 결과

#### 패턴 1: as const만 사용 ❌

```typescript
export const EMOTION_CANVAS_EFFECTS = {
  joy: { colors: ['#fff', '#000'] },
  sadness: { colors: ['#blue', '#gray'] }
} as const;

// 결과:
// - colors: readonly ['#fff', '#000'] (readonly 배열)
// - EmotionCanvasEffect의 colors: string[] 요구사항과 불일치
// - 에러 발생: readonly → mutable 할당 불가
```

#### 패턴 2: 타입 주석 사용 ⚠️

```typescript
export const EMOTION_CANVAS_EFFECTS: Record<string, EmotionCanvasEffect> = {
  joy: { colors: ['#fff', '#000'] },
  sadness: { colors: ['#blue', '#gray'] }
} as const;

// 결과:
// - typeof EMOTION_CANVAS_EFFECTS = Record<string, EmotionCanvasEffect>
// - keyof typeof EMOTION_CANVAS_EFFECTS = string (구체적 키 정보 손실)
// - 타입 주석이 as const의 추론을 오버라이드하여 에러 해결
```

#### 패턴 3: satisfies 사용 ✅

```typescript
export const EMOTION_CANVAS_EFFECTS = {
  joy: { colors: ['#fff', '#000'] },
  sadness: { colors: ['#blue', '#gray'] }
} satisfies Record<string, EmotionCanvasEffect>;

// 결과:
// - typeof EMOTION_CANVAS_EFFECTS = { joy: EmotionCanvasEffect, sadness: EmotionCanvasEffect }
// - keyof typeof EMOTION_CANVAS_EFFECTS = "joy" | "sadness" (정확한 키 보존)
// - colors는 string[]로 추론되어 조건 만족
```

### Q: satisfies는 어떻게 readonly 문제를 해결하나요?

**A**: `satisfies`에서 `as const`를 사용하지 않으면, 배열이 처음부터 `string[]`(mutable)로 추론됩니다. 

```typescript
const config = {
  colors: ['#f2d038', '#FFD700'] // 이미 string[]로 추론됨 (as const 없음)
} satisfies { colors: string[] };

// 과정:
// 1. colors가 string[]로 추론
// 2. satisfies가 string[]이 string[] 조건을 만족하는지 체크 ✅
// 3. 원본 타입 유지: colors는 string[]
```

### Q: as const satisfies 조합은 어떻게 작동하나요?

**A**: `as const satisfies` 패턴을 사용하면 더욱 강력한 타입 안전성을 얻을 수 있습니다:

```typescript
export const EMOTION_CANVAS_EFFECTS = {
  joy: { colors: ['#f2d038', '#FFD700'] },
  sadness: { colors: ['#268BD2', '#4682B4'] }
} as const satisfies Record<string, EmotionCanvasEffect>;

// 결과:
// - 각 키는 정확한 리터럴 타입으로 유지
// - readonly 배열이지만 구조적 호환성으로 string[] 조건 만족
// - 최고의 타입 안전성과 추론 정확성
```

**구조적 호환성**: TypeScript는 `readonly string[]`이 `string[]`와 구조적으로 호환된다고 판단하여 satisfies 검증을 통과시킵니다.

## 💡 실무에서의 선택 기준

### 언제 어떤 패턴을 사용해야 할까요?

| 패턴 | 사용 시기 | 장점 | 단점 |
|------|-----------|------|------|
| `satisfies` | **권장**: 대부분의 상황 | 키 정보 보존, 타입 안전성 | TypeScript 4.9+ 필요 |
| `타입 주석` | 레거시 프로젝트 | 모든 TS 버전 호환 | 키 정보 손실 |
| `as const만` | 읽기 전용 객체 | 최대 타입 엄격성 | 호환성 문제 |

### 실제 사용 예시

```typescript
// ✅ 최고의 패턴
export const API_ENDPOINTS = {
  users: { url: "/users", method: "GET" },
  posts: { url: "/posts", method: "POST" },
  comments: { url: "/comments", method: "GET" }
} satisfies Record<string, { url: string; method: string }>;

// 완벽한 타입 추론
type EndpointKeys = keyof typeof API_ENDPOINTS; // "users" | "posts" | "comments"
API_ENDPOINTS.users.url; // ✅ 타입: string
API_ENDPOINTS.unknownKey; // ❌ 컴파일 에러
```

## 🎯 결론

TypeScript의 `Record`, `satisfies`, `as const`를 조합하면:

1. **타입 안전성**: 컴파일 타임에 오류 방지
2. **정확한 추론**: 구체적인 키와 값 타입 보존  
3. **좋은 개발 경험**: 자동완성과 IntelliSense 활용

`satisfies` 연산자는 "검증은 하되 원본 정보 보존"이라는 개념으로 기존 패턴들의 한계를 보완했습니다. TypeScript 4.9 이상을 사용한다면 `satisfies` 패턴을 활용해보세요.

이러한 타이핑 기법을 통해 더 안전하고 유지보수하기 쉬운 TypeScript 코드를 작성할 수 있습니다.