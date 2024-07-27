---
layout: post
title: Yarn Berry PnP에서 Mac과 Docker 환경 동기화
description: >
  Docker를 통해 Yarn Berry(PnP) Zero Install을 구현하는 과정에서 발생한 환경 불일치 문제 해결 방법
sitemap: false
hide_last_modified: false
tags: [Yarn Berry, PnP, Zero Install, Docker, Mac, Cross-Platform Development]
keywords: [Yarn Berry, PnP, Zero Install, Docker, Mac, Cross-Platform Development]
---

# <span style="color: #0066cc;">Yarn Berry PnP Zero Install과 Docker 환경 동기화</span>

Yarn Berry PnP(Plug'n'Play)와 Zero Install을 활용한 백엔드 프로젝트를 Docker 환경으로 이전하는 과정에서 예상치 못하게 발생한 문제와 그 해결 과정을 기술 하였습니다.

> <small style="background-color: #f0f0f0; padding: 2px 5px; border-radius: 3px;">사용 버전: yarn-4.3.1.cjs, docker-24.0.2</small>

## <span style="color: #009688;">초기 문제 상황</span>

Docker 세팅을 마치고 컨테이너를 실행했을 때, 다음과 같은 에러 메시지가 출력되었습니다:

```terminal
oauth2.0-dev_1.0.0  | ! Corepack is about to download https://repo.yarnpkg.com/4.3.1/packages/yarnpkg-cli/bin/yarn.js
oauth2.0-dev_1.0.0  | 
oauth2.0-dev_1.0.0  | node:internal/modules/run_main:129
oauth2.0-dev_1.0.0  |     triggerUncaughtException(
oauth2.0-dev_1.0.0  |     ^
oauth2.0-dev_1.0.0  | Error: Required package missing from disk. If you keep your packages inside your repository then restarting the Node process may be enough. Otherwise, try to run an install first.
oauth2.0-dev_1.0.0  | 
oauth2.0-dev_1.0.0  | Missing package: cross-env@npm:7.0.3
oauth2.0-dev_1.0.0  | Expected package location: /usr/.yarn/berry/cache/cross-env-npm-7.0.3-96d81820f4-10c0.zip/node_modules/cross-env/
```

1. Corepack을 활성화하고 package.json에 Yarn 버전을 명시했기에 Corepack 관련 에러는 아니라고 가정
2. cross-env@npm:7.0.3 해당 특정 패키지를(존재함에도) 찾지못하는 이유가 내가 제공한 로컬 Yarn 패키지가 아닌 글로벌 Yarn을 사용한다고 판단

## <span style="color: #009688;">문제 해결 시도</span>


<span style="color: #009688;">첫 번째 해결 시도: 글로벌 캐시 비활성화</span>

글로벌 Yarn 사용을 막기 위해 .yarnrc.yml 파일에 다음 설정을 추가했습니다:

```yaml
enableGlobalCache: false
```

<br>


글로벌 캐시를 비활성화한 후, 새로운 에러가 발생했습니다:
```terminal
oauth2.0-dev_1.0.0  | Error [TransformError]: The package "@esbuild/linux-arm64" could not be found, and is needed by esbuild.
oauth2.0-dev_1.0.0  | 
oauth2.0-dev_1.0.0  | If you are installing esbuild with npm, make sure that you don't specify the
oauth2.0-dev_1.0.0  | "--no-optional" or "--omit=optional" flags. The "optionalDependencies" feature
oauth2.0-dev_1.0.0  | of "package.json" is used by esbuild to install the correct binary executable
oauth2.0-dev_1.0.0  | for your current platform.
```

1. @esbuild/linux-arm64를 찾지 못하는 이슈 발생
2. 해당 패키지의 로컬 존재 여부 확인 (존재하지 않음)
3. 왜 해당 패키지가 필요한지 고민

<span style="color: #009688;">두 번째 해결 시도: 크로스 플랫폼 지원 설정</span>

이 에러는 <span style="color: #FF5722;">Docker 환경(Linux)에 필요한 `@esbuild/linux-arm64` 패키지가 없어서</span> 발생했습니다. 로컬 환경(Mac)에는 `@esbuild-darwin-arm64`만 설치되어 있었기 때문입니다. 

Zero Install이 아닌 경우는 `yarn install` 명령어를 통해 의존성을 새로 설치하지만 <span style="color: #2196F3; font-weight: bold;">Zero Install은 로컬에 설치된 의존성을 그대로 가져가기 때문에</span> 도커 환경에서 <span style="color: #FF5722;">필요한 패키지가 누락</span>되는 문제가 발생했습니다.

Zero Install의 이점을 유지하면서 문제를 해결하기 위해, 로컬 환경에 Docker에서 필요한 운영체제용 패키지도 함께 설치하는 방법을 선택했습니다. `.yarnrc.yml` 파일에 다음 설정을 추가했습니다:

```yaml
enableGlobalCache: false
supportedArchitectures:
  os: [darwin, linux]
  cpu: [arm64]
```

```terminal
yarn install
```

이 설정으로 두 가지 운영체제(Mac, Linux) arm64 아키텍처에 따른 바이너리를 모두 로컬에 설치할 수 있게 되었습니다. `yarn install` 명령을 실행한 후, `.yarn/unplugged` 디렉토리에 Linux 바이너리가 설치된 것을 확인할 수 있었습니다.

## <span style="color: #009688;">주의사항</span>

<span style="color: #FF5722;">enableGlobalCache</span>를 <span style="color: #FF5722;">true</span>로 설정</span>하면, 추가적인 패키지 지원 설정에도 불구하고 오류가 발생할 수 있습니다. 이는 <span style="color: #2196F3;">일부 패키지가 특정 환경에서 완전히 지원하지 않을 수도  있기 때문</span>입니다.

## <span style="color: #009688;">결론</span>

이러한 접근 방식을 통해 Yarn Berry PnP의 Zero Install 기능을 유지하면서도 Mac 개발 환경과 Linux 기반 Docker 환경 간의 일관성을 확보 할 수 있었습니다. 이로써 <span style="color: #2196F3;">크로스 플랫폼 개발에서 발생할 수 있는 환경 불일치 문제를 효과적으로 해결</span>할 수 있었습니다.


