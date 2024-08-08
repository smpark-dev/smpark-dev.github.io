---
layout: post
title: NX에서 yarn berry pnp, node-modules 적용 시 발생하는 문제와 해결방법
description: >
   NX에서 yarn berry pnp, node-modules 적용 시 발생하는 문제와 해결방법 담을 포스팅
sitemap: true
image: /assets/img/sidebar-bg1.jpg
hide_last_modified: false
tags: [NX, Yarn Berry, PnP, 모노레포, TypeScript]
keywords: NX, Yarn Berry, PnP, 모노레포, TypeScript
categories: [dev, tools]
---

# <span style="color: #1976D2;">NX에서 yarn berry pnp, node-modules 적용 시 발생하는 문제와 해결방법</span>

NX 모노레포를 도입하면서 기존 프로젝트를 마이그레이션하는 과정에서 생긴 package 관리 이슈와 그 해결 과정을 기술하였습니다. 최대한 쉽고 자세히 쓰기 위해 노력했기 때문에 글이 장황할 수 있지만 이 글이 도움이 되었으면 좋겠습니다.

> <small style="background-color: #E3F2FD; padding: 2px 5px; border-radius: 3px;">사용 버전: yarn@4.4.0, nx@19.5.6</small>

## <span style="color: #1976D2;">목차</span>

1. [node-modules의 특징](#node-modules의-특징)
2. [PnP(Plug'n'Play)의 특징](#pnpplugnplay의-특징)
3. [문제 상황](#문제-상황)
4. [해결 방법](#해결-방법)
5. [결론](#결론)

## <span style="color: #1976D2;">node-modules의 특징</span>

기존의 node-modules 방식은 다음과 같은 특징이 있습니다:

- 패키지를 node-modules 폴더에 중첩된 폴더 구조로 설치되어 **<span style="color: #FF5722;">용량이 크다.</span>**
- 패키지를 선형으로 검색하면서 찾고 **<span style="color: #2196F3;">호이스팅</span>** 방식으로 찾을 수 있다.
- 내가 직접 설치하지 않은 패키지가 존재할 수 있는 **<span style="color: #FF5722;">유령 의존성</span>** 현상이 있다.

> <small style="background-color: #E3F2FD; padding: 2px 5px; border-radius: 3px;">호이스팅 방식이란</small>
>
> - <span style="color: #64B5F6;">**장점**</span>: node_modules의 공통 의존성을 디렉토리 상위로 이동시켜 중복 찾기를 방지
> - <span style="color: #F44336;">**단점**</span>: 버전 충돌 가능성과 유령 의존성 문제 발생 가능

> <small style="background-color: #E3F2FD; padding: 2px 5px; border-radius: 3px;">유령 의존성이란</small>
>
> - package.json에 명시되지 않은 패키지를 사용할 수 있게 되는 문제

## <span style="color: #1976D2;">PnP(Plug'n'Play)의 특징</span>

PnP 모드는 호이스팅을 사용하지 않고 다음과 같은 특징을 가집니다:

- 패키지를 중앙 저장소 **`pnp.cjs`**에 명시하고 **`.yarn/cache`**에 패키지를 압축(zip)해서 보관한다.
- 패키지를 찾을 때 **`pnp.cjs`**(Map & memory)에서 바로 찾는다. **<span style="color: #64B5F6;">시간복잡도 O(1)</span>**
- 패키지를 압축해서 보관하기 때문에 **<span style="color: #64B5F6;">용량이 작다.</span>**
- yarn install을 하지 않는 경제적인 **<span style="color: #64B5F6;">Zero Install</span>**이 가능하다.
- 패키지간 의존성 관리를 **<span style="color: #FF5722;">내가 직접 컨트롤해야 할 수도 있다.</span>**
- 패키지와의 호환성 문제로 해당 패키지를 포기해야 할 수도 있다.

## <span style="color: #1976D2;">문제 상황</span>

NX 모노레포를 처음 도입할 때, 어떤 방식의 패키지 매니저를 써야 할지 고민하게 될 수 있습니다.
저는 pnp를 선택했는데 그 과정에서 오는 문제점들 입니다.

### 1. 첫 번째 케이스

- **<span style="color: #FF5722;">pnp는 nx 시작 명령어 사용 불가. yarn nx 접두사만 가능.</span>** (하단코드 pnp, nx --version)

```terminal
smpark@MacBookAir smpark-monorepo % nx --version
Nx Version:
- Local: Not found
- Global: v19.5.6
smpark@MacBookAir smpark-monorepo % yarn nx --version
Nx Version:
- Local: v19.5.6
- Global: Not found
```

- node-modules의 경우 로컬과 글로벌 모두 버전 사용이 가능합니다. (nx 또는 yarn nx)
- **<span style="color: #64B5F6;">pnp는 로컬 NX만 사용 가능합니다.</span>**
- 사실 로컬에서 컨트롤하는 게 맞다고 생각하기에 문제라고 할 것까진 아닙니다.

> Why? 너는 되고 난 안 되나?
> - node-modules는 node_modules/.bin 디렉토리가 PATH 환경 변수에 자동으로 추가됩니다.
> - **<span style="color: #2196F3;">pnp는 .pnp.cjs 파일을 사용하여 모듈을 해결합니다. 전역 설치 개념이 없습니다.</span>**
> - pnp는 node_modules/.bin 디렉토리가 없어 PATH에 추가되지 않습니다.

### 2. 두 번째 케이스 (`workspaces 사용 시`)

```json
// package.json
"workspaces": [
  "apps/*"
],
```

- **<span style="color: #FF5722;">pnp는 package.json을 분리할 수 없다.</span>**
- node-modules는 Node.js의 모듈 해석 알고리즘으로 현재 디렉토리에서 시작해서 상위 디렉토리로 올라가며 node-modules를 찾기 때문에 루트의 공통 package.json과 각각 서브 프로젝트에 나누어져 있는 package.json으로 패키지가 나누어져 있어도 정상적으로 동작합니다.

> 즉, 루트에 typescript가 있고, 서브엔 없어도(typescript를 사용해야 하더라도) yarn nx serve your-subProject 같은 CLI를 입력하면 루트에 있는 typescript를 찾아서 정상 동작합니다.

- pnp는 루트의 pnp.cjs를 통해서 패키지를 찾습니다. 이렇게만 생각하면 당연히 루트에 pnp.cjs가 있으니 서브 프로젝트에서 typescript가 명시되어있지 않아도 루트에 명시되어 있으니 찾지 않을까? 싶지만 전혀 그렇지 않습니다. **<span style="color: #FF5722;">pnp는 명시적으로 선언된 의존성만 사용하기 때문입니다.</span>**

> 위에서 설명한 유령 의존성이 pnp에서 일어나지 않는 이유도 이것입니다. A패키지가 B패키지에 의존한다고 가정할 때, pnp는 A의 package.json에 명시된 B패키지만 사용하고 만약 제작자의 실수로 패키지B가 package.json에 명시되어 있지 않으면 사용하지 않습니다. 이 문제를 해결하기 위해 **<span style="color: #2196F3;">yarn.yml에 packageExtensions를 써서 사용자가 직접 연결시켜 주는 것입니다.</span>**

- 그 결과 pnp는 각각 프로젝트 별로 package.json을 따로 운용하는 방법과 모든 패키지를 root의 package.json에 기술하여 사용하는 방법으로 나뉘게 됩니다. 물론 packageExtensions를 써서 서브와 루트를 묶어주는 하이브리드 방법도 가능할 것 같긴 한데 너무 힘들 것 같네요 :(

> **<span style="color: #64B5F6;">저는 모든 패키지를 root의 package.json에 기술하는 방식으로 변경했습니다.</span>** 각각 서브의 package.json은 패키지 doc 같은 느낌으로 사용중입니다. 이 방식은 굳이 workspace가 필요 없기 때문에 삭제했습니다. root의 실행 컨텍스트에서 모든 프로젝트를 CLI를 통해 컨트롤 할 수 있기 때문입니다. 덕분에 각각 프로젝트의 project.json의 경로도 일정하게 관리가 가능합니다.

## <span style="color: #1976D2;">해결 방법</span>

1. **<span style="color: #64B5F6;">루트 package.json 활용</span>**: 모든 의존성을 루트 package.json에 명시하여 관리합니다. 이는 의존성 관리를 단순화하고 일관성을 유지할 수 있습니다.

2. **<span style="color: #64B5F6;">workspace 제거</span>**: PnP 환경에서는 workspace를 사용하지 않고도 효과적으로 모노레포를 관리할 수 있습니다.

3. **<span style="color: #64B5F6;">CLI 명령어 통일</span>**: `yarn nx` 접두사를 사용하여 모든 NX 관련 명령어를 실행합니다. 이는 로컬 NX 버전을 일관되게 사용할 수 있게 해줍니다.

4. **<span style="color: #64B5F6;">packageExtensions 활용</span>**: 필요한 경우 `.yarnrc.yml` 파일에 packageExtensions를 사용하여 패키지 간 의존성을 명시적으로 선언합니다.

## <span style="color: #1976D2;">결론</span>

NX 모노레포에서 Yarn Berry의 PnP를 사용할 때는 패키지 관리 방식에 주의가 필요합니다. node-modules와 달리 PnP는 더 엄격한 의존성 관리를 요구하지만, 이를 통해 더 안정적이고 예측 가능한 개발 환경을 구축할 수 있습니다.

**<span style="color: #64B5F6;">루트 package.json을 중심으로 의존성을 관리하고, CLI 사용 시 `yarn nx` 접두사를 일관되게 사용하는 것이 권장됩니다.</span>** 이러한 방식으로 NX와 Yarn Berry PnP의 장점을 최대한 활용하면서 효율적인 모노레포 관리가 가능해집니다.