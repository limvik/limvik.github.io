---
layout: post
title: 마음대로 정리해본 Spring Security Custom Authentication 절차 Diagram
categories:
- Spring
- Spring Security
tags:
- Spring Security
date: 2023-11-12 21:50 +0900
---
## Intro

Spring Security 를 사용하면서 개인적인 인증(Authentication) 절차를 만들 때 사용하는 큰 흐름을 나름대로 다이어그램으로 정리해봤습니다.

그런데 글씨가 많이 작습니다. 그래서 그림 해상도를 좀 높게 했는데, 그냥 가로로 그릴걸 후회됩니다.

## Diagram

Spring Security 팀에서 이미 구현해놓은 Authentication 인터페이스의 구현체는 보통 이름에 Token 이 붙습니다. AuthenticationConverter 인터페이스는 Spring Security 팀에서도 쓸 때도 있고, 안 쓸때도 있어서 취향 것 사용하시면 될거 같습니다.

![Spring Security Custom Authentication Diagram](/assets/img/2023-11-12-spring-security-custom-authentication-diagram/spring-security-custom-authentication-diagram.png)

## Outro

이렇게만 보면 참 간단해 보입니다. 너무 추상화했나 싶기도 하네요.
