---
layout: post
title: IntelliJ 에서 JUnit @DisplayName에 작성한 한글 깨짐 해결 방법
categories:
- Tools
tags:
- IntelliJ
- JUnit5
- Java
date: 2023-05-18 11:27 +0900
---
## Intro

책 실습을 따라하면서 IntelliJ를 사용해봤습니다. 그런데 JUnit 에서 테스트 이름을 설정하는 `@DisplayName` annotation에 한글을 사용하니 깨져서 나왔습니다.

![한글이 깨져서 보이는 화면](/assets/img/2023-05-18-cant-display-korean-junit-displayname-in-intellij/01.broken-character.png)

Encoding 설정 바꾸기, 한글 언어팩 설치, 구성 파일에서 VM 옵션 설정 등 여러가지 방법들을 모두 시도해봤는데 안돼서 당황했습니다.

쓸모없는 삽질 방지를 위해 기록 겸 공유드립니다.

## 환경

- IntelliJ IDEA 2023.1.2 (Community Edition)

`새로운 UI`를 적용해서 조금 달라보일 수는 있는데, 알아보시는데는 큰 무리 없을 것이라 판단됩니다.

## 간단한 설정 방법

방법은 간단합니다. `idea64.exe.vmoptions` 파일에 아래 vm 옵션을 추가해야 합니다.

```
-Dfile.encoding=UTF-8
```

다른 인코딩 설정은 다 `UTF-8` 아닌 걸로 설정하고 실험해보기도 하고, 다양한 방식으로 실험해 봤는데, 위 방법만 `@DisplayName` 에 작성한 한글이 출력됩니다.

![DisplayName에 작성한 한글이 이상 없이 출력된 화면](/assets/img/2023-05-18-cant-display-korean-junit-displayname-in-intellij/05.display-test-name-in-korean.png)

### idea64.exe.vmoptions 파일을 실행하고 옵션을 추가하는 방법

직접 IntelliJ가 설치된 디렉토리를 찾아가서 실행하는 방법도 있지만, 메뉴에서 실행하는게 더 간편합니다.

`도움말` 메뉴에서 `사용자 지정 VM 옵션 편집...` 을 선택하시거나,

![도움말 메뉴에서 사용자 지정 VM 옵션 편집을 선택하는 화면](/assets/img/2023-05-18-cant-display-korean-junit-displayname-in-intellij/02.vmoptions-in-help-menu.png)

`shift` 키를 두 번 눌러 `vmoptions` 또는 메뉴 이름의 일부인 `VM 옵션 편집` 등으로 검색하여 `사용자 지정 VM 옵션 편집...` 을 선택합니다.

![shift 키를 두 번 눌러 사용자 지정 VM 옵션 편집을 선택하는 화면](/assets/img/2023-05-18-cant-display-korean-junit-displayname-in-intellij/03.double-click-shift.png)

그럼 아래와 같이 idea64.exe.vmoptions 파일이 실행되는데, 아래와 같이 `-Dfile.encoding=UTF-8` 를 추가합니다.

![idea64.exe.vmoptions 파일에서 VM 옵션을 추가한 화면](/assets/img/2023-05-18-cant-display-korean-junit-displayname-in-intellij/04.add-option-to-idea64.exe.vmpoptions-file.png)

## 중요한 것은

가장 중요한 것은 `IntelliJ를 종료했다가 다시 실행`하는 겁니다.

근데 저는 종료했다가 다시 실행해도 안돼서 굉장히 당황스러웠는데, IntelliJ 프로젝트를 2개 켜놔서 모두 종료했다가 다시 실행 하니까 됩니다.

종료했다가 다시 실행 했는데도 안되시는 분들은 `IntelliJ가 2개 이상 열려있는지 확인하시고, 모두 종료 후 다시 실행`한 후에 테스트를 실행해보시기 바랍니다.

## Outro

새로운 툴을 사용하니까 별거 아닌 문제에 시간을 허비하게 되는 것 같습니다.

저처럼 어이없는 곳에서 시간 허비하지 마시고, IntelliJ가 모두 종료됐는지 잘 확인하셔서 시간 아끼세요!
