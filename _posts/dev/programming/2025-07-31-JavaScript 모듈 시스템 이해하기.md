---
layout: post
title: "JavaScript 모듈 시스템: import와 export 이해하기"
description: "ES6 모듈 시스템의 핵심 개념과 실무 활용법"
sitemap: true
image: /assets/img/sidebar-bg3.jpg
hide_last_modified: false
tags: [programming, javascript, module, es6]
keywords: programming, javascript, module, es6
categories: [dev, programming]
---

JavaScript 모듈 시스템은 코드를 구조화하고 재사용 가능하게 만드는 강력한 도구입니다. `import`와 `export` 키워드를 사용하여 모듈 간 기능을 공유하고, 대규모 애플리케이션을 효율적으로 관리할 수 있습니다.

## 📤 Export 유형

### Default Export

모듈당 하나의 기본 내보내기를 허용합니다. 주로 모듈의 주요 기능이나 컴포넌트를 내보낼 때 사용합니다.

```javascript
// HomePage.js
export default function HomePage() {
  return <div>홈페이지 컴포넌트</div>;
}

// 또는 별도로 선언 후 내보내기
function HomePage() {
  return <div>홈페이지 컴포넌트</div>;
}
export default HomePage;
```

### Named Export

여러 값을 이름을 지정하여 내보낼 수 있습니다. 유틸리티 함수나 상수 등을 그룹화할 때 유용합니다.

```javascript
// utils.js
export const formatDate = (date) => {
  return new Intl.DateTimeFormat('ko-KR').format(date);
};

export const API_URL = 'https://api.example.com';

export function calculateAge(birthDate) {
  const today = new Date();
  const birth = new Date(birthDate);
  return today.getFullYear() - birth.getFullYear();
}
```

### 혼합 사용

Default Export와 Named Export를 함께 사용할 수도 있습니다:

```javascript
// math.js
export default function add(a, b) {
  return a + b;
}

export const PI = 3.14159;
export const E = 2.71828;
```

## 📥 Import 방식

### Default Import

기본 내보내기를 가져올 때는 중괄호 없이 원하는 이름을 사용할 수 있습니다.

```javascript
import HomePage from './HomePage';
import MyComponent from './HomePage'; // 다른 이름으로도 가능
```

### Named Import

명명된 내보내기를 가져올 때는 중괄호를 사용합니다.

```javascript
import { formatDate, API_URL } from './utils';

// 별칭 사용
import { formatDate as dateFormatter } from './utils';

// 선택적 가져오기
import { calculateAge } from './utils';
```

### 네임스페이스 Import

모든 named export를 하나의 객체로 가져올 수 있습니다:

```javascript
import * as utils from './utils';

// 사용 시
const formattedDate = utils.formatDate(new Date());
console.log(utils.API_URL);
```

### 혼합 Import

기본 내보내기와 명명된 내보내기를 동시에 가져올 수 있습니다.

```javascript
import React, { useState, useEffect } from 'react';
import add, { PI, E } from './math';
```

### 동적 Import

런타임에 모듈을 비동기적으로 로드할 수 있습니다:

```javascript
// 조건부 로딩
if (condition) {
  const module = await import('./conditionalModule');
  module.default();
}

// 코드 스플리팅
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

## 🔄 Re-export

다른 모듈에서 가져온 항목을 즉시 다시 내보내는 방식입니다. 라이브러리나 패키지에서 API를 정리할 때 유용합니다.

### Default Export Re-export

```javascript
// index.js - 메인 컴포넌트를 재내보내기
export { default } from './HomePage';
export { default as HomePage } from './HomePage';
```

### Named Export Re-export

```javascript
// index.js - 여러 유틸리티를 한 곳에 모으기
export { HomePage, AboutPage } from './components';
export { formatDate, API_URL } from './utils';
export { validateEmail, validatePhone } from './validators';
```

### 전체 Re-export

```javascript
// 모든 named export 재내보내기
export * from './utils';

// 네임스페이스로 재내보내기
export * as components from './components';
```

### Named to Default Re-export

```javascript
// 특정 named export를 default로 재내보내기
export { HomePage as default } from './HomePage';
```

## 💡 모범 사례와 팁

### 1. 모듈당 하나의 책임

각 모듈은 하나의 주요 기능이나 관련 기능 그룹을 담당하게 하세요.

```javascript
// ❌ 좋지 않은 예 - 너무 많은 책임
// userUtils.js
export const formatUserName = () => {};
export const validateEmail = () => {};
export const fetchUserData = () => {};
export const renderUserProfile = () => {};

// ✅ 좋은 예 - 책임 분리
// userFormatter.js
export const formatUserName = () => {};

// userValidator.js  
export const validateEmail = () => {};

// userAPI.js
export const fetchUserData = () => {};
```

### 2. 명확한 네이밍

내보내기와 가져오기 시 명확하고 설명적인 이름을 사용하세요.

```javascript
// ❌ 모호한 이름
import { fn1, data } from './utils';

// ✅ 명확한 이름
import { formatDate, API_BASE_URL } from './utils';
```

### 3. 일관된 Export 스타일

프로젝트 전체에서 일관된 export 스타일을 사용하세요.

```javascript
// 함수형 컴포넌트는 default export
export default function Button() {}

// 여러 유틸리티는 named export
export const utils = {
  format: () => {},
  validate: () => {}
};
```

### 4. 순환 의존성 피하기

모듈 간 순환 참조를 피해 코드의 복잡성을 줄이세요.

```javascript
// ❌ 순환 의존성
// moduleA.js
import { funcB } from './moduleB';

// moduleB.js  
import { funcA } from './moduleA';

// ✅ 공통 모듈로 분리
// shared.js
export const sharedFunc = () => {};

// moduleA.js
import { sharedFunc } from './shared';

// moduleB.js
import { sharedFunc } from './shared';
```

### 5. 트리 쉐이킹 활용

사용하지 않는 코드를 제거하기 위해 named export를 적극 활용하세요.

```javascript
// ✅ 트리 쉐이킹 친화적
export const utilA = () => {};
export const utilB = () => {};
export const utilC = () => {};

// 사용하는 곳에서
import { utilA } from './utils'; // utilB, utilC는 번들에 포함되지 않음
```

### 6. 타입과 함께 사용하기 (TypeScript)

TypeScript를 사용한다면 타입도 함께 내보내세요.

```typescript
// types.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export type UserRole = 'admin' | 'user' | 'guest';

// userService.ts
import type { User, UserRole } from './types';

export const createUser = (userData: User): User => {
  return userData;
};
```

## 마무리

JavaScript 모듈 시스템은 현대 웹 개발에서 중요한 부분입니다. 이 글에서 정리한 내용들이 도움이 되었으면 좋겠습니다.

모듈 시스템을 잘 활용하면 코드의 재사용성을 높이고 팀 협업에도 도움이 됩니다. 처음에는 익숙하지 않을 수 있지만, 조금씩 사용해보시면서 본인만의 패턴을 찾아가시면 될 것 같습니다.
