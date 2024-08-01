---
layout: post
title: NX 모노레포 적용 시리즈[NodeJS, NextJS, Yarn](2)
description: >
  NX 모노레포의 기본 세팅 방법을 소개하는 포스팅입니다.
sitemap: true
image: /assets/img/sidebar-bg.jpg
hide_last_modified: false
tags: [NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub]
keywords: NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub, 모노레포
categories: [dev, tools]
---

# NX 모노레포 기본 세팅하기

## 0. 사전 준비 및 설치

NodeJS, 패키지 매니저(NPM(5.2.0+)[npx, yarn]), Git, IDE(VSCODE), Terminal
위의 기본 요구사항이 준비되어있으면 설치할 수 있습니다. 

```bash
// npx create-nx-workspace을 하면 로컬 설치는 기본적으로 적용
//local
npm install -D nx@latest

// global
npm install -g nx@latest
```

1. NX 생성
[NX 공식 문서 링크](https://nx.dev/getting-started/intro?utm_medium=website&utm_campaign=homepage_links&utm_content=cta_hero_get_started#try-nx-yourself)

NX 모노레포, 줄여서 NX는 2가지 적용 방법을 제공합니다.

1. <span style="color: skyblue;">NX 워크스페이스를 최초로 생성하는 방법</span>
```bash
  # terminal
  npx create-nx-workspace
```


2. <span style="color: skyblue;">기존 워크스페이스에 NX를 추가하는 방법</span>

```bash
  # terminal
  npx nx init
```

저는 가지고 있는 다양한 기술 스택의 개인 프로젝트들을 하나의 모노레포 워크스페이스에 넣어서 관리하기 위해 1번 방식을 채택하였습니다.

3. NX 생성 과정 (위와 동일한 스타일)
![NX 설치 과정 1](/assets/img/blog/nx-install1.png)
- NX를 생성할 폴더의 위치를 정하고 기술 스택을 고릅니다. 
저는 하나의 기술 스택을 베이스로 여러 프로젝트를 관리하는 것이 아닌
다양한 기술 스택의 프로젝트를 관리할 예정이기 때문에 None을 선택했습니다. 

![NX 설치 과정 2](/assets/img/blog/nx-install2.png)
- NX 워크스페이스의 구조와 관리 방식의 선택입니다. (최초 선택한 메뉴마다 상이할 수 있습니다.)
1) 패키지 기반(package-based)
특징:

각 프로젝트가 독립적인 npm 패키지로 취급됨
각 프로젝트는 자체 package.json 파일을 가짐
프로젝트 간 의존성은 npm 패키지처럼 관리됨

```tree
my-workspace/
├── package.json
├── nx.json
├── packages/
│   ├── app1/
│   │   ├── package.json
│   │   └── src/
│   └── lib1/
│       ├── package.json
│       └── src/
````

사용 사례: 여러 독립적인 라이브러리나 애플리케이션을 개발하면서 각각을 npm에 게시하고자 할 때 유용합니다.

2) 통합(integrated)
특징:

모든 프로젝트가 단일 리포지토리에서 관리됨
루트 레벨의 단일 package.json 파일 사용
프로젝트 간 의존성이 내부적으로 관리됨

```tree
my-workspace/
├── package.json
├── nx.json
├── apps/
│   └── app1/
│       └── src/
├── libs/
│   └── lib1/
│       └── src/
````

사용 사례: 밀접하게 연관된 여러 애플리케이션과 라이브러리를 개발할 때 적합합니다. 예를 들어, 프론트엔드와 백엔드, 그리고 공유 라이브러리를 함께 개발하는 경우입니다.

3) 독립형(standalone) 

특징:

단일 프로젝트에 NX의 기능을 적용
기존 프로젝트를 NX로 마이그레이션하기 쉬움
프로젝트 구조가 가장 단순함

```tree
my-project/
├── package.json
├── nx.json
├── src/
└── tests/
```

사용 사례: 단일 애플리케이션이나 라이브러리 프로젝트에 NX의 빌드 최적화, 캐싱, 코드 생성 등의 기능을 활용하고 싶을 때 적합합니다.

자신에게 맞는 방식을 채택하시면 됩니다. 
저는 프론트, 백 코드가 섞여있기 때문에 통합형을 선택했습니다. 

![NX 설치 과정 3](assets/img/blog/nx-install3.png)

마지막으로 사용하실 CI 도구를 선택하시면 됩니다. 
저는 GitHub에 프로젝트를 올리고 배포할 예정이기에 GitHub Actions를 선택했습니다.


이렇게 기본 설치 방법이 끝났습니다. 다음 포스팅에서는 적용 방법에 대해서 작성하겠습니다.