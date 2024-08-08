---
layout: post
title: NX 모노레포 적용 시리즈 [NodeJS, NextJS, Yarn, PnP] (3)
description: >
  NX 모노레포의 기본 세팅 방법(NodeJS)을 소개하는 포스팅입니다.
sitemap: true
image: /assets/img/sidebar-bg3.jpg
hide_last_modified: false
tags: [NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub]
keywords: NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub, 모노레포, PnP
categories: [dev, tools]
---

# NX 모노레포 적용 시리즈 [NodeJS, NextJS, Yarn, PnP] (3)

## 📚 목차

1. [Yarn PnP vs Node Modules](#1-yarn-pnp-vs-node-modules)
2. [NodeJS 프로젝트 준비하기](#2-nodejs-프로젝트-준비하기)
3. [Package.json Scripts로 구동하기](#3-packagejson-scripts로-구동하기)
4. [Project.json 작성 및 NX CLI로 구동하기](#4-projectjson-작성-및-nx-cli로-구동하기)
5. [Production과 Build 설정](#5-production과-build-설정)

## 1. Yarn PnP vs Node Modules

🤔 어떤 패키지 관리 방식을 선택해야 할까요?

자세한 내용은 [NX Package Manager](https://smpark-dev.github.io/dev/tools/2024-07-26-NX에서-yarn-berry-pnp,-node-modules-적용-시-발생하는-문제와-해결방법.html) 포스팅을 확인해주세요!

> 💡 Tip: 초기 설치 시 기본적으로 node-modules가 설정되어 있습니다.

## 2. NodeJS 프로젝트 준비하기

1. 📁 NX root에 `apps` 폴더 생성
2. 🚚 프로젝트를 `apps/your-project`로 이동 (git clone, 드래그앤드롭, 복사 붙여넣기 등 편한 방법으로)
3. 🧹 불필요한 파일 제거:
   - `.vscode` (root의 .vscode로 관리)
   - `node_modules` (root에서 관리 시 제거)
   - `.pnp.cjs`, `.pnp.loader.mjs`, `.yarn` (root에서 관리 시 제거)
   - `yarn.lock` (root에서 관리 시 제거)
   - `.yarnrc.yml` (root에 없다면 root로 이동 또는 `yarn set version berry`로 새로 설치)
4. ✍️ `apps/your-project/project.json` 생성

   ```json
   // project.json
   {
     "name": "smpark-oauth2.0",
     "version": "1.0.0",
     "private": true,
     "sourceRoot": "apps/smpark-oauth2.0",
     "projectType": "application",
     "tags": []
   }
   ```

   > name을 최초에 잘 정의해두셔야 CLI를 할 때 잘 찾아갑니다.

## 3. Package.json Scripts로 구동하기

NX 모노레포에는 두 가지 실행 방법이 있습니다:

1. 기존 package.json의 scripts 사용
2. 새로 생성한 project.json의 target 사용

먼저 package.json scripts를 사용해 보겠습니다.

1. 📝 Root `package.json`에 workspace 설정 추가:

```json
"workspaces": [
  "apps/*"
]
```

2. 🖥️ 실행 명령어:

💡 **Node-modules 모드**:

- `yarn install` 후 바로 실행 가능

💡 **PnP 모드**:

- `yarnrc.yml`을 `nodeLinker: pnp`로 변경
- `yarn install` 실행
- 실행 컨텍스트가 서브 프로젝트의 package.json을 참조하므로 문제 없이 실행됨

```bash
// yarn nx <your cli> <your-project-name>
yarn nx dev smpark-oauth2.0
```

💡 **실행 결과**:

```
> nx run smpark-oauth2.0:dev

[nodemon] 3.1.4
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src/**/*.ts src/**/*.js
[nodemon] watching extensions: ts,js
[nodemon] starting `eslint && tsx watch --tsconfig tsconfig.json ./src/index.ts`
Connected to MongoDB!
Server Connected, 4000 port!
```

3. 🔄 문제 발생 시 캐시 리셋:

```bash
yarn nx reset
yarn install
```

> 사실 이렇게 workspace를 통해 monorepo를 쓸 거 였으면 nx를 도입 할 이유가 없습니다. 이제 nx를 이용한 세팅 방식으로 넘어가 보겠습니다.

## 4. Project.json 작성 및 NX CLI로 구동하기

이제 project.json을 작성하고 NX CLI를 통해 프로젝트를 실행해보겠습니다.

📝 `project.json` 예시:

```json
{
  "name": "smpark_oauth2.0",
  "version": "1.0.0",
  "private": true,
  "sourceRoot": "apps/smpark_oauth2.0",
  "projectType": "application",
  "tags": [],
  "targets": {
    "dev": {
      "executor": "nx:run-commands",
      "options": {
        "command": "cross-env NODE_ENV=development tsx watch --tsconfig apps/smpark_oauth2.0/tsconfig.json apps/smpark_oauth2.0/src/index.ts"
      }
    }
  }
}
```

> `dev` CLI로 접근 후 command를 실행 합니다. 각각의 파일에 접근하기 위해 apps/smpark_oauth2.0/ 경로를 적용하려는 파일 앞에 추가 작성하였습니다.

🖥️ NX CLI로 실행하기:

🔍 **Node-modules 모드**:

- `yarn install` 후 바로 실행 가능
- root package.json에 공통 패키지, sub package.json에 특정 패키지 관리 가능
- workspaces 설정 유지 필요

🔍 **PnP 모드**:

- 서브 프로젝트의 package.json 패키지 리스트를 root package.json으로 이동 필요
- `yarn install`로 종속 패키지 설치 후 실행
- workspaces 제거 가능 (apps/your-project에서 yarn 명령어 사용 불가)
- 실행 컨텍스트가 root로 변경되므로 프로젝트 내 경로에 `apps/your-project` 추가 필요할 수 있음

> node-modules, pnp의 차이가 아닌, package.json의 분할 여부로 workspaces 사용이 정해짐

```bash
yarn nx serve smpark_oauth2.0
```

💡 **실행 결과**:

```
> nx run smpark_oauth2.0:serve

> cross-env NODE_ENV=development tsx watch --tsconfig apps/smpark_oauth2.0/tsconfig.json apps/smpark_oauth2.0/src/index.ts

Connected to MongoDB!
Server Connected, 4000 port!
```

## 5. Production과 Build 설정

🛠️ Production 실행과 Build를 위한 설정 추가:

```json
"prod": {
  "executor": "@nx/js:node",
  "defaultConfiguration": "production",
  "dependsOn": ["build"],
  "options": {
    "buildTarget": "smpark-oauth2.0:build"
  },
  "configurations": {
    "development": {
      "buildTarget": "smpark-oauth2.0:build:development"
    },
    "production": {
      "buildTarget": "smpark-oauth2.0:build:production"
    }
  }
},
"build": {
  "executor": "@nx/esbuild:esbuild",
  "outputs": ["{options.outputPath}"],
  "defaultConfiguration": "production",
  "options": {
    "main": "src/index.ts",
    "outputPath": "dist",
    "outputFileName": "apps/smpark-oauth2.0/index.js",
    "tsConfig": "apps/smpark-oauth2.0/tsconfig.json",
    "format": ["esm"],
    "bundle": true,
    "assets": [
      {
        "glob": "**/*",
        "input": "apps/smpark-oauth2.0/src/views",
        "output": "apps/smpark-oauth2.0/src/views"
      },
      {
        "glob": "**/*",
        "input": "apps/smpark-oauth2.0/src/public",
        "output": "apps/smpark-oauth2.0/src/public"
      },
      {
        "glob": "**/*",
        "input": "apps/smpark-oauth2.0/src/script",
        "output": "apps/smpark-oauth2.0/src/script"
      }
    ],
    "platform": "node"
  },
  "configurations": {
    "development": {
      "minify": false
    },
    "production": {
      "minify": true
    },
    "define": {
      "process.env.NODE_ENV": "'production'"
    }
  }
}
```
> prod는 build한 폴더를 통해 실행하도록 설정
> assets을 통해 빌드시에 옮겨지지 않는 폴더를 복사 후 이동 설정

💡 **주의사항**:

- esbuild를 사용하므로 root에 `@nx/esbuild` 설치 필요
- assets를 통해 빌드 시 복사되지 않는 폴더 설정 가능

🌟 NX 설정의 장점:

- 정교한 캐싱 가능
- 상세한 빌드 컨트롤 가능
- 의존성 그래프 확인 가능
- 병렬 실행 최적화 가능

다음 포스팅에서는 NextJS 프로젝트의 NX 적용에 대해 알아보겠습니다! 👋
