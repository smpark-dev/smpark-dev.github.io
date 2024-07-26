---
layout: page
title: Config
description: >
  이 챕터는 Hydejack의 다양한 구성 옵션을 다루며, 이를 통해 필요에 맞게 조정할 수 있습니다.
hide_description: true
sitemap: false
---

Jekyll이 실행되면 `_config.yml`에 다양한 항목을 추가하여 기본 구성을 시작할 수 있습니다. 
여기 문서 외에도 [주석이 달린 설정 파일][config]을 읽을 수 있습니다.

`_config.yml`을 변경할 때, 변경 사항이 적용되도록 Jekyll 프로세스를 다시 시작해야 합니다.
{:.note}

0. 이 무작위 시드 목록은 무작위 목록으로 toc로 대체됩니다.
{:toc}

## `url` 및 `baseurl` 설정
첫 번째로 해야 할 일은 `_config.yml` 파일에서 올바른 `url` 및 `baseurl` 값을 설정하는 것입니다.

`url`은 사이트의 도메인으로, 프로토콜(`http` 또는 `https`)을 포함합니다. 이 사이트의 경우:

~~~yml
# 파일: `_config.yml`
url: https://qwtel.com
~~~

GitHub Pages 또는 Netlify에 호스팅할 때는 이 속성을 제공할 필요가 없습니다.
{:.note}

Jekyll 블로그가 페이지의 하위 디렉토리에 호스팅되는 경우, `/`로 시작하고 `/`로 끝나지 않는 경로를 `baseurl`에 제공하세요. 예를 들어:

~~~yml
# 파일: `_config.yml`
baseurl: /hydejack
~~~

그렇지 않으면 빈 문자열 `''`을 제공합니다.

GitHub Pages 또는 Netlify에 호스팅할 때는 이 속성을 제공할 필요가 없습니다.
{:.note}

### GitHub Pages
[GitHub Pages](https://pages.github.com/)에 호스팅할 때, `url`은 `https://<username>.github.io`입니다
(사용자 정의 도메인을 사용하는 경우 제외).

`baseurl`은 호스팅하는 페이지 유형에 따라 다릅니다.

* *사용자 또는 조직 페이지*를 호스팅할 때는 빈 문자열 `''`을 사용하세요.
* *프로젝트 페이지*를 호스팅할 때는 `/<reponame>`을 사용하세요.

GitHub에서 호스팅할 수 있는 페이지 유형에 대한 자세한 내용은 [GitHub 도움말 기사](https://help.github.com/articles/user-organization-and-project-pages/)를 참조하세요.

## 강조 색상 및 사이드바 이미지 변경
Hydejack은 사이드바의 배경 이미지와 강조 색상(링크 색상, 선택 및 포커스 윤곽 등)을 선택할 수 있게 합니다.

~~~yml
# 파일: `_config.yml`
accent_image: /assets/img/sidebar-bg.jpg
accent_color: rgb(79,177,186)
~~~

텍스트가 읽기 쉬운 상태로 유지되도록 흐릿한 이미지를 사용하는 것이 좋습니다. 흐릿한 이미지를 JPG로 저장하면 파일 크기도 크게 줄어듭니다.
{:.note}

`accent_image` 속성은 기본 이미지를 제거하는 특별한 값 `none`도 허용합니다.

사이드바 이미지에 밝은 색상이 포함된 경우, 흰색 텍스트를 읽기 어려울 수 있습니다. 이 경우, `invert_sidebar: true`를 프론트 매터에 설정하여 사이드바의 텍스트 색상을 반전시키는 것을 고려해 보세요.
[프론트 매터 기본값][fmd]을 사용하여 모든 페이지에서 이를 활성화할 수 있습니다(아래 참조).

이 값들은 페이지별로 덮어쓸 수 있으며, 각 페이지에 독특한 외관을 만들 수 있습니다.
또한 [프론트 매터 기본값][fmd]을 통해 카테고리의 모든 게시물에 특정 외관을 적용할 수 있습니다. 예를 들어:

```yml
# 파일: `_config.yml`
defaults:
  - scope:
      path:         hydejack/
    values:
      accent_image: /assets/img/hydejack-bg.jpg
      accent_color: rgb(38,139,210)
