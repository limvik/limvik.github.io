---
layout: post
title: Youtube 요약 - Spring 개발자의 흔한 실수
categories:
- Spring
tags:
- Spring
date: 2024-02-11 19:25 +0900
---
## Intro

Youtube [SpringDeveloper](https://www.youtube.com/@SpringSourceDev) 채널의 Spring Office Hours 팟캐스트 내용([Spring Office Hours S3E4 - Common Spring Developer Mistakes](https://www.youtube.com/watch?v=nd5JzDIEI6A))을 일부 추려서 정리한 글입니다. 가벼운 분위기에서 떠들면서 하는 이야기라 생략과 의역을 많이 했습니다.

## 1위. 모든 것을 Public으로 만들기

흔히 `레이어별 패키지(package by layer)` 방식으로 어플리케이션을 만들면서 Controller, Service 등의 레이어 간의 상호작용을 위해 모든 접근 권한을 public 으로 만듭니다. 하지만, 이것은 옳은 방법이 아닙니다.

이를 방지하기 위한 방법의 한 예는 `기능별 패키지(package by feature)`를 만드는 것입니다. product, order 등의 패키지를 만들고 normal 한 접근 권한을 부여합니다. 일부 패키지 간의 의사소통이 필요할 수 있지만, 모든 것을 public으로 만들지는 않아도 됩니다.

## 2위. Field Injection

DI(Dependency Injection)를 할 때 `Field Injection`이 선택지에 없었으면 좋겠습니다.

Field Injection을 하면 테스트 시에 Mocking 하기 어려운 것과 같은 문제가 있습니다. Field Injection 보다는 `Constructor Injection`을 사용하는 것이 낫습니다.

`Field Injection` 이 허용되는 곳은 테스트 뿐입니다. 테스트를 위한 테스트를 작성하지는 않기 때문입니다.

## 번외. 기술 선택/신규 기능에 대한 이야기

### 기술 선택

채팅으로 올라온 using Data JPA when all you need is Data JDBC 에 대한 응답

기술을 선택할 때는 팀의 강점도 생각해야합니다. 팀원 대부분이 Hibernate를 이해하지 못하고, 내부적으로 어떻게 동작하는지 잘 모르는데 당신이 JPA를 좋아한다고 JPA를 선택하면 그것은 잘못된 결정입니다.

만약 당신의 서비스 10개 중 9개에서 JPA를 사용하고 있다면,  작고 사소한 것이라도 그냥 JPA를 사용해도 상관 없습니다.

### 신규 기능

Spring Boot 최신 버전에 새로운 기능이 나왔을 때, 이 새로운 기능을 사용하기 위해 전체 어플리케이션을 다시 작성해야 한다는 생각을 버려야합니다.

새로운 기능은 당신의 사이드 프로젝트에 적용해 보세요.

## 3위. 모든 것에 인터페이스 만들기

Java의 인터페이스와 그 구현이 1개 있는 것을 본적 있을 겁니다. 모든 것이 인터페이스와 그 구현을 가질 필요는 없습니다.

이론적으로는 훌륭할지 몰라도, 실제로 구현이 교체될 확률은 없습니다. 구체적인 계획이 없다면 Type으로 시작하세요.

이와 연관되어 spring.start.io 나 spring CLI에서 프로젝트를 시작하는 것도 흔한 실수입니다. 일주일 동안 프로젝트에서 어떠한 가치도 만들어내지 못한채 Outline과 Boiler Plate 만 작성하고 있어서는 안됩니다.

## 기타 이야기

- `@Transactional` 잘 이해하고 써야합니다.
- REST API를 만들 때는 일반적인 규칙 따라야합니다.
- SRP(Single Responsibility Principle)을 따라야합니다. 예를들어, Controller는 요청을 받고 응답하는 용도로 사용하고, 로직을 포함해서는 안됩니다.
- 한 클래스에 모두 밀어넣기의 반대편에는 과도한 설계가 있습니다. 사회자는 개인적으로 일단 작업하고, 어떻게 더 좋게만들지 고민하며 리팩터링하는 것을 선호합니다.
- 테스트를 작성하세요.
- Java와 Spring 버전을 업그레이드하여 성능과 보안상의 이점을 얻고, 새로운 기능을 사용하세요. Record 클래스 강추
- 예외 처리는 필수 입니다.
- 문서화를 잘 해야합니다.
- 데이터 유효성 검사를 수행하세요.
- Spring Actuator는 결국에 추가할 수 밖에 없을 겁니다.
- Spring에서 Slice 테스트를 하지 않으면, 테스트가 너무 커져서 오래걸릴 수 있습니다.

## Outro

바닥에 흘린 물과 같이 깊이는 없지만, 수행했던 프로젝트를 돌아볼 수 있는 시간이었습니다.

두서없이 가볍게 이야기하는 팟캐스트라 이에 대한 비판이 있었는데, 깊이있는 내용을 원하면 사회자 중 한 분인 [Dan Vega](https://www.youtube.com/@DanVega)의 채널을 보라고 합니다.

Spring Office Hour는 처음 봤는데, 다른 주제를 보게 되더라도, 아주 가벼운 마음으로 보는게 좋겠습니다... 하하...
