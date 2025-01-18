---
title: "Editorconfig"
summary: "프로젝트에서 간단한 컨벤션 규칙을 정의할 수 있는 Editorconfig에 대해서 알아보자."
date: 2022-06-07
---

여러 명이 같은 프로젝트를 다루다 보면 서로의 코딩 스타일이 달라 섞이는 경우가 많다.
코드 규칙을 유지하기 위해서 다양한 방법들이 탄생했는데, Editorconfig는 그 중 아주 간단한 방법이다.

## Editorconfig?

Editorconfig는 `.editorconfig`라는 설정 파일로 사용하는 간단한 컨벤션 방식이다.

Editorconfig 자체는 컨벤션 규칙의 스펙을 정의할 뿐이고,
각각의 IDE나 에디터들이 프로젝트 디렉토리에 있는 `.editorconfig` 파일을 해석해 코딩 스타일을 적용한다.
일부는 자체적으로 지원하고, 일부는 플러그인의 형태로 지원한다.
예를 들어 Intellij IDEA는 IDE 자체적으로 Editorconfig를 지원하고, Visual Studio Code는 [공식 플러그인](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) 형태로 지원한다.

자세한 지원 사항은 [공식 홈페이지](https://editorconfig.org/#pre-installed)에 정리되어 있다.

## .editorconfig 파일

Editorconfig는 `.editorconfig` 파일 하나로 사용한다.
`.editorconfig` 파일은 적용하고 싶은 루트 디렉토리에 위치하면 된다.
아래와 같은 형태로 작성하면 된다.

```editorconfig
# 반드시 root = true 로 시작해야 한다.
root = true

# 모든 파일에 적용
[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
tab_width = 4
insert_final_newline = true
trim_trailing_whitespace = true
max_line_length = 80

# 확장자가 md인 파일에 적용
[*.md]
max_line_length = off

# 여러 개의 확장자에 동시 적용
[*.{json,yaml,yml,toml}]
indent_size = 2
tab_width = 2

# 특정 경로, 특정 파일 패턴에 적용
[.github/workflows/**.{yaml,yml}]
indent_size = 2
tab_width = 2

# 특정 파일에 적용
[package.json]
indent_size = 4
tab_width = 4
```

## 속성

Editorconfig의 표준 속성은 총 7개가 있다.

| 속성                       | 타입                     | 설명                          |
|--------------------------|------------------------|-----------------------------|
| charset                  | string                 | 파일의 인코딩 방식을 지정. (`utf-8` 등) |
| end_of_line              | `lf` or `cr` or `crlf` | 한 줄의 끝을 나타내는 문자 형식.         |
| indent_style             | `space` or `tab`       | 들여쓰기 방식.                    |
| indent_size              | int                    | 들여쓰기 크기.                    |
| tab_width                | int                    | 탭 크기.                       |
| insert_final_newline     | boolean                | 파일 마지막을 빈 줄로 끝낼지 말지 여부.     |
| trim_trailing_whitespace | boolean                | 후행 공백문자 제거 여부.              |
| max_line_length          | int or `off`           | 한 줄의 최대 크기. `off`인 경우 무한대.  |

표준 속성들은 아주 기초적인 컨벤션만 정의한다.
그 때문에 철저한 코딩 스타일을 유지하기에는 부족한 면도 있다.
하지만 반대로 생각하면 최소한의 규칙만 정하고 사용하기에는 아주 가볍고 편리하기도 하다.

이 외에도 특정 IDE가 자기만의 속성을 정의하는 경우가 있다.
그런 경우 상당히 세세하게 컨벤션을 지정할 수 있고, 이를 파일 하나로 저장해 프로젝트 참여 인원 전원이 공통적인 스타일을 가져갈 수도 있다.
다만, 이런 특수 속성들은 해당 에디터가 아니면 지원하지 않는 경우가 많다.

## 문법

반드시 `root = true`로 시작해야 한다.
그리고 그 뒤부터 파일에 대한 규칙을 정한다.

```editorconfig
[파일 패턴]
속성
```

기본적으로 위처럼 `[]`에 적용하고 싶은 파일을 지정하고 그 아래에 적용할 속성을 적는다.
`[*]`를 사용해 모든 파일에 일괄 적용할 수도 있고, `[filename.txt]`처럼 특정 파일만 지정할 수도 있다.
`[dir/test/**.md]`와 같이 특정 경로도 지정 가능하고, `[*.java]`처럼 특정 확장자만 지정할 수도 있다.
만약 여러 확장자에 동시에 적용하고 싶다면 `[*.{json,yml,toml}]`처럼 작성해야 한다.

더 자세한 규칙은 [공식 웹사이트](https://editorconfig.org/#file-format-details)에서 볼 수 있다.

## 총평

개인적으로 요즘 일부 프로젝트에 조금씩 써보고 있는데 (예를 들면 이 블로그) 나름 만족스럽다.
보통 엄격한 컨벤션 툴이나 린트 툴들은 설정이 꽤나 복잡한데, 간단한 파일 하나로 기초적인 설정이 가능하는 점이 마음에 든다.
어느 파일에나 적용 가능할 만한 기초적인 부분만 다루기 때문에 범용적이라는 장점도 있다.

다만, 엄격한 코딩 스타일을 적용하고 싶을 때는 부족하다.
요즘 코딩 스타일을 유지할 방법에 대해 찾아보고 있는데 소스 코드에 쓰기에는 부족한 것 같다.
아무래도 Editorconfig로 범용적인 파일 컨벤션을 잡고, 소스 코드는 더 엄격한 툴을 사용해야 할 것 같다.

## 참고

- [Editorconfig Official Website](https://editorconfig.org/)
