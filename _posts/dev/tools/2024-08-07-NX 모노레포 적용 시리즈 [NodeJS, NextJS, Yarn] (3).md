---
layout: post
title: NX 모노레포 적용 시리즈 [NodeJS, NextJS, Yarn, PnP] (3)
description: >
  NX 모노레포의 기본 세팅 방법(NodeJS)을 소개하는 포스팅입니다.
sitemap: true
image: /assets/img/sidebar-bg.jpg
hide_last_modified: false
tags: [NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub]
keywords: NX, MonoRepo, Yarn, NodeJS, NextJS, GitHub, 모노레포, PnP
categories: [dev, tools]
---

# <span style="color: skyblue;">NX 모노레포 적용 시리즈 \[NodeJS, NextJS, Yarn\] (3)</span>

## 0. yarn pnp와 yarn node-modules 무엇을 쓸 것 인가?

이 주제는 해당 포스팅을 참고해주시기 바랍니다.
[NX Package Manager](https://smpark-dev.github.io/dev/tools/2024-07-26-NX%EC%97%90%EC%84%9C-yarn-berry-pnp,-node-modules-%EC%A0%81%EC%9A%A9-%EC%8B%9C-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EB%AC%B8%EC%A0%9C%EC%99%80-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95.html)

> 처음 설치 시 기본적으로 node-modules가 설정되어있다.

## 1. NodeJS 프로젝트 마이그레이션 및 기본 준비하기.

패키지 매니저 설정이 완료되었다고 가정하고 NodeJS 프로젝트를 NX 모노레포에 적용해보겠습니다.

- 먼저 전 포스팅에서 생성한 nx의 root에 apps 폴더를 생성합니다.
- 그리고 해당 apps에 프로젝트를 옮깁니다. 깃 클론, 드래그앤드롭, 복사 붙여넣기 등 편한대로 옮겨주세요. 그럼 경로가 다음과 같이 됩니다. `apps/your-project`
- 이제 필요없는 파일을 제거해 줍니다. 예를들면 :
  - .vscode: root의 .vscode로 관리합니다.
  - node-modules: root에서 관리한다면 제거합니다. 각자 패키지를 관리한다면 지울 필요 없습니다.
  - .pnp.cjs, .pnp.loader.mjs, .yarn: 위와 동일 합니다.
  - yarn.lock: 위와 동일 합니다.
  - .yarnrc.yml를 사용하신다면 해당 파일이 root의 없으면 root에 생성 또는 옮겨서 사용해주세요. `yarn set version berry` 같은 명령어로 새로 설치하셔도 됩니다.
    > 저는 패키지를 루트에서 관리한다는 가정하에 설명합니다.
- 추가한 프로젝트의 root에 project.json을 생성해주세요. `apps/your-project/project.json`
- project.json은 각 서브 프로젝트마다 1개씩 존재합니다. 이를 통해 각 프로젝트의 CLI, cache, options들을 컨트롤 할 수 있습니다. 자세한 설정은 공식문서를 참고해주세요.

## 2. NodeJS 프로젝트 package.json scripts로 구동하기.

기본 작성은 공식문서를 찾아보시거나 직접 자신의 환경에 맞는 nx를 생성하시고 그곳에 존재하는 디폴트 값을 참조하셔도 됩니다. 다음은 저의 project.json 입니다.

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

package.json과 매우 유사합니다.
일단 위 단계로만 자신의 프로젝트를 정의해주세요. name을 최초에 잘 정의해두셔야
CLI를 할 때 잘 찾아갑니다.

---

nx monorepo에는 2가지 실행 방법이 있습니다.

- 기존의 package.json의 scripts에 적힌 CLI를 실행하는 방법
- 새로 project.json를 생성하고 target을 작성하여 실행하는 방법입니다.

nx monorepo는 CLI를 실행하면 project.json의 target을 먼저 찾고 그 다음 package.json의 스크립트를 찾아 실행합니다.

먼저 package.json의 스크립트를 통해서 프로젝트를 실행해보겠습니다. 다만 이를 위해서는 apps안의 프로젝트들을 하나의 워크스페이스로 묶어야 합니다.

```json
  // root/package.json
  "workspaces": [
    "apps/*"
  ]
```

```terminal
  // terminal
  // yarn nx <your cli> <your-project-name>
  yarn nx dev smpark-oauth2.0
```

그럼 이제 실행해보겠습니다.

만약 node-modules를 사용한다면 그냥 `yarn install` 후에 아래처럼 하면, 해당 프로젝트에 문제가 없는 이상 잘 작동합니다.
만약 다 세팅했는데도 CLI이 먹히지 않을땐 아래 명령어를 이용해 캐시를 리셋해주세요.

```terminal
// terminal
yarn nx reset
yarn install
```

```terminal
// terminal
smpark@MacBookAir practice-monorepo % yarn nx dev smpark_oauth2.0
> nx run smpark_oauth2.0:dev

[nodemon] 3.1.4
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src/**/*.ts src/**/*.js
[nodemon] watching extensions: ts,js
[nodemon] starting `eslint && tsx watch --tsconfig tsconfig.json ./src/index.ts`
Connected to MongoDB!
Server Connected, 4000 port!
```

실행이 잘되네요.

---

그럼 이제 pnp모드로 해보겠습니다.
`yarnrc.yml`을 nodeLinker: pnp로 바꾸고 다시 `yarn insatll` 합니다.
그리고 다시 CLI 입력.

```terminal
 // terminal
 smpark@MacBookAir practice-monorepo % yarn nx dev smpark_oauth2.0
 > nx run smpark_oauth2.0:dev

 [nodemon] 3.1.4
 [nodemon] to restart at any time, enter `rs`
 [nodemon] watching path(s): src/**/*.ts src/**/*.js
 [nodemon] watching extensions: ts,js
 [nodemon] starting `eslint && tsx watch --tsconfig tsconfig.json ./src/index.ts`
 Connected to MongoDB!
 Server Connected, 4000 port!
```

잘 됩니다.

그런데 ?? 뭔가 이상합니다. 왜 잘 될까요? pnp는 root에서 실행했는데? pnp의 동작 매커니즘을 생각하면 root의 package.json은 서브 프로젝트의 패키지 중 아무것도 설치되어 있지 않아서 패키지를 찾을 수 없다고 해야하는데? 어떻게 이게 가능할까요?

그건 바로, root에서 실행한 CLI가 서브 프로젝트의 package.json의 스크립트를 탔기 때문입니다. 실행 컨텍스트가 root에서 서브 프로젝트로 넘어간거죠. 그렇기 떄문에 실제로는 서브 프로젝트에서 `yarn dev`를 한 것과 같은 효과가 발생하여 해당 프로젝트의 package.json을 바라보게 된 것이고 문제 없이 실행이 된겁니다. 물론 workspace 세팅이 되어있기 때문이기도 합니다.

하지만 이렇게 workspace를 통해 monorepo를 쓸 거 였으면 nx를 도입 할 이유가 없습니다. 이제 nx를 이용한 세팅 방식으로 넘어가 보겠습니다.

## 3. NodeJS 프로젝트 project.json 작성하기.

저는 다음과 같이 작성해주었습니다. 
```json
// project.json

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

`dev` CLI로 접근할 거고 커맨트를 통해 실행 합니다. 각각의 파일에 접근하기 위해 apps/smpark_oauth2.0/ 경로를 적용하려는 파일 앞에 추가 작성해 줬습니다. 

만약 nodeLinker: node-modules 로 변경되었다면 yarn install 후에 바로 정상 작동 합니다. 

```terminal
// terminal
smpark@MacBookAir practice-monorepo % yarn nx serve smpark_oauth2.0

> nx run smpark_oauth2.0:serve

> cross-env NODE_ENV=development tsx watch --tsconfig apps/smpark_oauth2.0/tsconfig.json apps/smpark_oauth2.0/src/index.ts

Connected to MongoDB!
Server Connected, 4000 port!
```

아까보다 출력이 깔끔해졌네요. 
node-modules를 이용하면 root package.json에 공통 패키지, sub package.json에 특정 패키지를 두어서 나누어서 관리할 수 있습니다. 단 이렇게 관리할 시 workspaces는 유지해주어야 합니다.

다음은 pnp입니다. 

pnp는 위의 node-modules처럼 실행하면 특정 패키지를 찾을 수 없다는 에러를 만나게 됩니다. 왜냐하면 workspace로 묶여있어도 root에서 CLI를 실행했고, 서브 프로젝트의 package.json을 타지도 않아서 실행 컨텍스트가 root가 됩니다. 즉, root의 package.json을 바라보게 됩니다. 
그리고 현재 root의 package.json은 텅텅 비어있죠. 이를 실행 시키려면 서브의 package.json의 패키지 리스트를 root의 package.json으로 복붙해 옮겨야 합니다. 

얼추 중복 없이 옮기고 난 후에 `yarn install`로 종속된 패키지들을 설치하고 실행합니다.

```
// terminal
smpark@MacBookAir practice-monorepo % yarn nx serve smpark_oauth2.0

> nx run smpark_oauth2.0:serve

> cross-env NODE_ENV=development tsx watch --tsconfig apps/smpark_oauth2.0/tsconfig.json apps/smpark_oauth2.0/src/index.ts

Connected to MongoDB!
Server Connected, 4000 port!
```

잘 작동하는군요. 모든 종속성을 root에 옮겼기 때문에 workspace는 이제 필요 없습니다. workspace를 제거하면 apps/your-project로 이동 후에 yarn 명령어를 쓸 수 없게 됩니다. 또 서브의 package.json은 doc나 다름 없는 신세가 되었습니다. 물론 조금의 트릭을 사용해서 pnp의 root에서 서브 프로젝트의 package.json을 바라보게 실행하는 방법도 있습니다만 추천하지 않습니다.

또 한가지, 실행 컨텍스트가 root로 바뀌었기 때문에 옮긴 프로젝트 안에서 사용되던 경로가 있다면 해당 경로에 apps/your-project를 추가해주어야 할 수 있습니다. 


마지막으로 prod와 build입니다. 
prod는 build한 폴더를 통해 실행하도록 만들었습니다. 
assets을 통해 빌드시에 옮겨지지 않는 폴더를 복사해서 옮길 수 있게 설정했습니다. 
esbuild를 사용해서 빌드했기 때문에 root에 @nx/esbuild를 설치해야 합니다.

```json
// project.json
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
},
```

지금은 간단하게 실행만 했지만 NX를 사용하면 정교한 캐싱, 상세한 빌드 컨트롤, 의존성을 그래프로 확인, 병렬 실행 최적화 등등을 이용할 수 있습니다. 

다음 포스팅은 nextjs 프로젝트의 NX 적용 포스팅 입니다.