---
layout: post
title: Spring Security (9) ProviderManager
categories:
- Framework
- Spring
- Spring Security
tags:
- Spring
- Spring Security
date: 2023-07-29 20:56 +0900
---
## Intro

AuthenticationManager 에 대해서 다뤘던 [이전 글](/posts/study-spring-security-8-authenticationmanager/)에 이서 [ProviderManager](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/authentication/ProviderManager.html) 를 살펴보겠습니다.

## ProviderManager

### 이전 글에서 살펴본 Provider Manager

이전 글 마지막에 문서([링크](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-providermanager))를 살펴보며 ProviderManager는 AuthenticationManager 의 가장 흔히 사용되는 구현이고, ProviderManager는 AuthenticationProviders 에 위임한다는 것을 아래 그림과 함께 확인했습니다.

![ProviderManager 와 AuthenticationProviders 다이어그램](/assets/img/2023-07-24-study-spring-security-8-AuthenticationManager/03-providermanager-and-authentication-providers.png)

### ProviderManager의 전체적인 동작 흐름

이어서 나오는 ProviderManager 관련 내용을 살펴보면, 아래와 같습니다.

>Each `AuthenticationProvider` has an opportunity to indicate that authentication should be successful, fail, or indicate it cannot make a decision and allow a downstream `AuthenticationProvider` to decide. If none of the configured `AuthenticationProvider` instances can authenticate, authentication fails with a `ProviderNotFoundException`, which is a special `AuthenticationException` that indicates that the `ProviderManager` was not configured to support the type of `Authentication` that was passed into it.

각 AuthenticationProvider는 인증의 성공, 실패, 보류 여부를 결정하거나 다음 AuthenticationProvider가 결정하게 할 수 있다고 합니다. (Filter 생각나게 하는 패턴입니다.) 그리고 인증 대상인 Authentication 객체가 ProviderManager에 있는 AuthenticationProvider 중에 인증을 할 수 있는 AuthenticationProvider가 없으면 `AuthenticationException` 중에 하나인 `ProviderNotFoundException` 과 함께 인증에 실패한다고 합니다.

이전에 Authentication 과 관련된 글([링크](https://limvik.github.io/posts/study-spring-security-7-securitycontext-authentication-grantedauthority/))에서 봤던 Authentication 객체에 저장된 정보(Credentials, Principals)를 가지고, 인증 작업을 수행하는 구체적인 과정을 AuthenticationProvider에서 수행할 것으로 유추해볼 수 있습니다.

문서에서도 바로 뒤에 각 AuthenticaionProvider 가 특정 유형의 인증을 수행하는 방법을 알고 있고, 특정 유형의 구체적인 예로는 username/password 의 검증이나 SAML assertion 을 인증을 들고 있습니다.

### ProviderManager가 여러 개인 경우

AuthenticationManager Bean만 노출한 채로 여러 AuthenticationProvider 를 통해 여러 유형의 인증을 지원할 수 있다는 언급도 하고 있습니다.

![ProviderManager의 Parent 다이어그램](/assets/img/2023-07-29-study-spring-security-9-ProviderManager/01-providermanager-parent.png)

갑자기 또 ProviderManager의 부모인 AuthenticationManager 를 언급하는 이유는, 현재 ProviderManager 내 에서 인증을 수행할 수 있는 AuthenticationProviders 가 없는 경우 다른 AuthenticaionManager 의 구현을 참조하도록 설정할 수 있기 때문입니다. 이와 같은 경우 아래 그림과 같은 구조가 됩니다. 하지만 Option일 뿐 강제사항은 아닙니다.

![여러 개의 ProviderManagers와 Parent 다이어그램](/assets/img/2023-07-29-study-spring-security-9-ProviderManager/02-providermanagers-parent.png)

다음과 같이 ProviderManager([Github 링크](https://github.com/spring-projects/spring-security/blob/main/core/src/main/java/org/springframework/security/authentication/ProviderManager.java#L104-L130))에는 parent AuthenticationManager 는 선택적으로 사용할 수 있도록 Constructor 가 정의되어 있습니다.

```java
// ProviderManager.java
public ProviderManager(AuthenticationProvider... providers) {
    this(Arrays.asList(providers), null);
}

public ProviderManager(List<AuthenticationProvider> providers) {
    this(providers, null);
}

public ProviderManager(List<AuthenticationProvider> providers, AuthenticationManager parent) {
    Assert.notNull(providers, "providers list cannot be null");
    this.providers = providers;
    this.parent = parent;
    checkState();
}
```

문서에서는 일부 `인증(공유된 부모 AuthenticationManager)`은 공통적이지만 인증 `메커니즘(다른 ProviderManager 인스턴스)`이 다른 여러 SecurityFilterChain([관련 글](/posts/study-spring-security-4-securityfilterchain-and-filters/)) 인스턴스가 있는 시나리오에서 ProviderManager가 여러개 인 것이 어느 정도 일반적인 경우라고 합니다.

![SecurityFilterChains 다이어그램](/assets/img/2023-07-08-study-spring-security-4-SecurityFilterChain-and-Filters/03-more-spread-securityfilterchain.png)

### 주의 사항

문서에서 ProviderManager 의 마지막 문단을 살펴보겠습니다.

> 기본적으로 ProviderManager는 성공적인 인증 요청에 의해 반환되는 Authentication 객체에서 민감한 자격 증명(credentials) 정보를 지우려고 시도합니다. 이렇게 하면 비밀번호와 같은 정보가 HttpSession에서 필요 이상으로 오래 유지되는 것을 방지할 수 있습니다.

그리고 이렇게 Credentials를 제거하는 경우, `캐시`를 사용할 때 문제가 됨을 언급하고 있습니다. 구체적인 예로는 `stateless 앱`에서 사용자 객체 정보(문서에서 UserDetails 를 예로들었는데 이 객체는 아직 나온적 없으므로, 나중에 관련 정보를 추가하겠습니다.)를 캐시해놓고 사용할 때 Credentials가 지워져 버린 경우 더 이상 캐시된 값에 대해 인증할 수 없는 상황을 이야기합니다.

그리고 이러한 문제의 해결책으로, 아래 두 가지를 제시합니다.

1. cache implementation 또는 반환된 Authentication 객체를 생성하는 AuthenticationProvider 에서 객체의 복사본 만들기
2. ProviderManager에서 eraseCredentialsAfterAuthentication 속성 비활성화

1번의 경우 아직 와닿는게 없습니다만, Credentials 를 갖고 있는 Authentication 객체를 복사해두고 인증 시에 사용하는 것으로 생각됩니다. 자세한 정보를 얻으면 추후에 추가하겠습니다.

2번 같은 경우 간단하게 메서드를 호출해서 설정할 수 있습니다.

```java
public void setEraseCredentialsAfterAuthentication(boolean eraseSecretData) {
    this.eraseCredentialsAfterAuthentication = eraseSecretData;
}
```

그런데 굳이 1번처럼 복잡하게 하기보다는 간단하게 2번을 호출하는게 개인적으로는 더 좋아보입니다.

### Event Publishing

ProviderManager의 API 문서([링크](https://docs.spring.io/spring-security/site/docs/6.1.2/api/org/springframework/security/authentication/ProviderManager.html))를 보면, Event Publishing 에 관한 이야기가 나옵니다.

AuthenticationEventPublisher 에 대해서 문서에서 별도로 다루고 있어([링크](https://docs.spring.io/spring-security/reference/servlet/authentication/events.html)), 때가 되면 별도의 글로 작성해 보겠습니다.

### 정리

ProviderManager 도 구체적인 인증 절차는 AuthenticationProvider 로 넘기기 때문에, 그냥 이런게 있구나 하면서 가볍게 살펴봤습니다.

전체적인 흐름을 러프하게 정리해보면 ProviderManager 가 Authentication 객체를 받아서 해당 Authentication을 처리할 수 있는 AuthenticationProvider 로 넘겨 인증 절차를 수행합니다.

1. 인증에 성공한 경우: ProviderManager는 기본적으로 Credentials 를 지우고 Authentication 객체를 반환하므로, stateless 앱에서 캐시를 사용하는 경우 setEraseCredentialsAfterAuthentication 메서드로 인증에 사용할 Credentials를 지우지 않게 하는 등의 원활한 인증을 위한 조치를 취해주어야 합니다.
2. 인증에 실패한 경우: AuthenticationException 을 던집니다. 만약 ProviderManager에 있는 AuthenticationProvider 중에 입력된 Authentication 을 인증할 수 있는 AuthenticationProvider가 없다면, ProviderNotFoundException 을 던집니다. 이때 여러 개의 ProviderManager 를 사용하면서, 동일한 Parent AuthenticationManager를 갖는 경우 parent를 설정하여 다른 ProviderManager 로 인증 절차를 넘길 수도 있습니다.
3. 판단할 수 없는 경우: 다음 AuthenticationProvider로 넘길 수 있습니다.

## Outro

다음에 살펴봐야 할 AuthenticationProvider 는 문서 상에 설명이 두 문장 밖에 없지만, 지금 사용하고 있는 AuthenticationProvider 를 구체적으로 살펴보고 싶어서 다음 글로 넘기려고 합니다.

그럼 다음은 실제 인증을 수행하는 AuthenticationProvider 를 살펴보겠습니다. 