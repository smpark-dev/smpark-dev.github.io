---
layout: post
title: NX에서 Yarn Berry PnP 적용 시 패키지 문제 해결
description: >
  NX 모노레포를 적용하는 과정에서 node-modules가 아닌 pnp를 적용시 패키지가 호이스팅 되지 않아 생긴 문제를 해결하는 포스팅
sitemap: false
hide_last_modified: false
tags: [NX, Yarn Berry, PnP, 모노레포, TypeScript]
keywords: [NX, Yarn Berry, PnP, 모노레포, TypeScript]
---

# <span style="color: #0066cc;">NX 모노레포에서 Yarn Berry PnP 적용하기</span>

NX 모노레포를 도입하면서 기존 프로젝트를 마이그레이션하는 과정에서, `root/package.json`을 공통으로 두고 각 프로젝트의 `package.json`을 서브로 설정하기로 했습니다. 이 과정에서 모든 프로젝트가 루트의 Yarn Berry PnP를 바라보게 하는 과정에서 생긴 이슈와 그 해결 과정을 기술 하였습니다.

> <small style="background-color: #f0f0f0; padding: 2px 5px; border-radius: 3px;">사용 버전: yarn-4.3.1.cjs, nx-19.5.3</small>

## <span style="color: #009688;">패키지 호이스팅의 이해</span>

Node.js의 npm과 yarn 같은 패키지 매니저들은 기본적으로 패키지 호이스팅을 수행합니다:

- <span style="color: #4CAF50;">**장점**</span>: node_modules의 공통 의존성을 디렉토리 상위로 이동시켜 중복 설치를 방지
- <span style="color: #F44336;">**단점**</span>: 버전 충돌 가능성과 유령 의존성 문제 발생 가능
> <small style="background-color: #FFFDE7; padding: 2px 5px; border-radius: 3px;">유령 의존성이란 package.json에 명시되지 않은 패키지를 사용할 수 있게 되는 문제</small>

## <span style="color: #009688;">PnP(Plug'n'Play) 방식의 특징</span>

PnP 방식은 호이스팅을 사용하지 않고 다음과 같은 특징을 가집니다:

- `.pnp.js` 파일로 의존성 정보 관리
- 중앙 저장소(예: `.yarn/cache`)에 파일 저장 
- 모듈 위치 즉시 파악으로 인한 빠른 속도
- 중앙 캐시 사용으로 설치 및 빌드 시간 대폭 감소

## <span style="color: #009688;">문제 상황</span>

NX 모노레포에 PnP 방식을 적용하여 TypeScript를 공통(root) 패키지에 두고, 서브 패키지에는 TypeScript를 명시하지 않으려 했으나, <span style="color: #F44336;">호이스팅 방식이 아니어서 서브 패키지에서 TypeScript를 찾지 못하는 문제</span>가 발생했습니다.

## <span style="color: #009688;">해결 방법: Yarn Berry의 제약 조건 시스템 활용</span>

Yarn Berry의 제약 조건(constraints) 시스템을 사용하여 Prolog 스크립트로 문제를 해결했습니다.

### <span style="color: #00796B;">스크립트의 목적</span>
1. 모노레포 내 모든 프로젝트의 TypeScript 버전 일관성 유지
2. 루트 프로젝트에서 버전을 한 번만 정의하고 모든 서브 프로젝트에 자동 적용
3. TypeScript를 항상 개발 의존성(devDependencies)으로 처리
4. <span style="color: #FFA000;">다만, 이 방식은 `모든 서브프로젝트에 동일한 버전의 공통 패키지가 강제`된다는 점에 유의해야 합니다.</span>

### <span style="color: #00796B;">적용 방법</span>

1. root에 `constraints.pro` 파일 생성 및 코드 삽입:

<pre style="padding: 10px; border-radius: 5px;">
<code>% 공통 패키지 목록 정의
common_packages(['typescript']).

% 의존성 규칙 설정
gen_enforced_dependency(WorkspaceCwd, Package, Version, DependencyType) :-
  % 공통 패키지 목록 가져오기
  common_packages(Packages),
  
  % 현재 패키지가 공통 패키지 목록에 있는지 확인
  member(Package, Packages),
  
  % 루트 워크스페이스에서 패키지 버전 가져오기
  workspace_has_dependency('.', Package, RootVersion, _),
  
  % 모든 워크스페이스에 패키지 추가
  (
    % 루트 워크스페이스인 경우 원래 버전 유지
    WorkspaceCwd == '.' -> Version = RootVersion
    % 서브 프로젝트인 경우 루트의 버전으로 설정
    ; Version = RootVersion
  ),
  
  % 의존성 타입 설정 (예: devDependencies로 통일)
  DependencyType = devDependencies.</code>
</pre>

2. 터미널에 명령어 실행 
<pre style="padding: 10px; border-radius: 5px;">
<code>// constraints.pro 파일에 정의된 규칙에 위배되는 사항을 알린다.
yarn constraints 
// constraints.pro 파일에 정의된 규칙에 위배되는 사항을 고친다.
yarn constraints --fix</code>
</pre>

결과 `root/package.json`에 설치된 Typescript가 `sub/package.json`에도 같은 버전으로 
작성되는걸 볼 수 있습니다. 

<span style="padding: 5px; border-radius: 3px;">만약 위와 같은 방식을 채택하지 않는다면 root에서 추가되는 모든 패키지를 서브에도 작성해주어야 하기 때문에 
불편해집니다. 위와 같은 방식으로 패키지를 일관되게 관리하고 편의성을 얻을 수 있습니다.</span>