---
layout: post
title: 첫 오픈소스(Spring Security) 기여 경험
categories:
- Git
- Github
tags:
- Git
- Github
- Spring Security
date: 2023-08-09 12:42 +0900
---
## Intro

Spring Security 를 공부하면서 Contribution 을 해보고 싶다는 생각이 들어서 약 한 달전에 처음으로 PR(Pull Request)를 올렸었는데 한국 시간 기준으로는 어제 드디어 merge가 됐습니다.

코드 가독성 개선을 위한 PR([링크](https://github.com/spring-projects/spring-security/pull/13472))과 공식 문서 링크 수정한 PR([링크1](https://github.com/spring-projects/spring-security/pull/13565), [링크2](https://github.com/spring-projects/spring-security/pull/13593))을 올렸습니다. 문서 링크 수정한 PR 1개 빼고 3개 중에 2개는 merge가 됐습니다.

처음에는 너무 사소해서, 이런거 올리면 괜히 사람들 귀찮게 만드는거 아닐까 싶어서 주저하다가 올렸었습니다. 저 처럼 뭔가 주저하시는 분들에게 도움이 될까 싶어서 과정을 공유드립니다.

## Contribution Guideline 살펴보기

Spring Security 의 Contribution Guideline([링크](https://github.com/spring-projects/spring-security/blob/main/CONTRIBUTING.adoc))을 살펴보면, 굉장히 친절해서 읽으면서 용기를 얻게 됩니다.

> Please do your best to follow these steps. Don’t worry if you don’t get them all correct the first time, we will help you.

절차를 몇개 빼먹어도 도와줄거라고 이렇게 친절하게 이야기하고 있습니다.

사소한 거는 그냥 issue 올리지 말고, 바로 PR을 올리라고 써있어서 부담없이 PR 을 올리기로 결심했습니다.

> Must you create an issue first? No, but it is recommended for features and larger bug fixes. It’s easier discuss with the team first to determine the right fix or enhancement. For typos and straightforward bug fixes, starting with a pull request is encouraged. Please include a description for context and motivation. Note that the team may close your pull request if it’s not a fit for the project.

## 첫 PR

처음에 SecurityFilterChain을 공부하면서 FilterOrderRegistration([Github 링크](https://github.com/spring-projects/spring-security/blob/61bb9ab938a7e6c05012138fdd775ff736256b59/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java#L130)) 클래스에 개선할 것이 보였습니다(현재는 제 코드가 반영되었습니다).

기존에는 filter 를 추가할 때 containsKey 메서드를 사용해서 filter가 존재하는지 확인하고, put 을 해서 추가하는 방식이었습니다.

```java
void put(Class<? extends Filter> filter, int position) {
    String className = filter.getName();
        if (this.filterToOrder.containsKey(className)) {
            return;
        }
        this.filterToOrder.put(className, position);
}
```

기본서를 보면서 Map 메서드 중에 Java 8 에서 추가된 putIfAbsent([API 문서 링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.html#putIfAbsent%28K,V%29)) 메서드를 알고 있었기 때문에, 코드 가독성 측면에서 개선을 할 수 있겠다 싶었습니다.

containsKey 후에 put을 하는 것보다 putIfAbsent 메서드를 사용하는게 성능적인 측면에서도 조금은 좋다고는 하지만, filter 갯수 자체가 몇개 안되고 한정적이라 성능적인 측면에서는 큰 의미 없을거라 생각해서 가독성 측면을 강조하기로 계획했습니다.

```java
void put(Class<? extends Filter> filter, int position) {
    this.filterToOrder.putIfAbsent(filter.getName(), position);
}
```

너무 사소한거 같아 약간 걱정은 되었지만 첫 PR을 올렸습니다.

하지만 몇 일동안 피드백이 전혀 없어서, 아 너무 사소해서 무시당했나 싶었습니다. 그래서 조금 더 올려서 양으로 성의를 보일겸, 다른 사람들이 PR을 올리고 merge 된 것들을 참고해보기로 합니다.

## 다른 사람 PR 찾아보고 따라하기

이미 merge 돼서 closed 된 것 중에 [Using modern java features](https://github.com/spring-projects/spring-security/pull/12569) 가 눈에 보였습니다. 

그중에서도 Java16에서 정식으로 도입([JEP 394](https://openjdk.org/jeps/394))된 instanceof 를 pattern matching 방식([Oracle 문서](https://docs.oracle.com/en/java/javase/17/language/pattern-matching-instanceof-operator.html#GUID-843060B5-240C-4F47-A7B0-95C42E5B08A7))으로 사용해서 코드 가독성을 개선할 수 있는 것이 눈에 들어왔습니다.

> java에 pattern matching 적용 방향에 대해 다룬 문서([링크](https://openjdk.org/projects/amber/design-notes/patterns/pattern-matching-for-java))를 보면 단순히 가독성 개선을 위한 것은 아니란걸 알 수 있습니다. pattern matching 전문가가 되기에는 아직 시기상조라 생각해서 자세히 읽어보진 않았습니다...

pattern matching 사용방법을 간단히 살펴본 후 몇 개의 클래스를 수정하고 commit을 했는데 문제가 생겼습니다.

당시에 git과 github 사용이 지금보다 훨씬 미숙해서, 처음 putIfAbsent 메서드로 수정했던 브랜치와 같은 브랜치에서 작업하고 commit을 하면 동일한 PR 에 commit이 올라간다는 사실을 모르고, 같은 브랜치에 commit을 했습니다.

첫 PR([링크](https://github.com/spring-projects/spring-security/pull/13459))의 제목이 Refactor put method for readability in FilterOrderRegistration 이었는데, 제목하고 맞지 않게 돼버려서 자체적으로 폐기했습니다.

이때 Github PR 에 대해서도 배우고, 작업을 위한 브랜치를 따로 만드는게 좋다는 점을 배울 수 있었습니다.

### 새로운 PR 작성

putIfAbsent 메서드를 사용한 코드와 instanceof 를 pattern matching 방식으로 사용한 코드가 포함된 새로운 PR([링크](https://github.com/spring-projects/spring-security/pull/13472))을 작성하면서 조금 더 추상화된 제목을 작성했습니다. 

> Refactor for readability

그리고 신나서 올렸는데, 또 몇일 동안 응답이 없었습니다...

여전히 뭔가 부족한가 싶어서 주석의 오타 같은 것도 수정하면서 양을 늘렸습니다. 그래도 계속되는 무응답에 초보자가 코드를 건들면 안되는건가 싶어서, 다른 사람들의 PR을 또 살펴보다보니 공식 문서의 오타(typo) 같은 아주 사소한 것도 올리고 merge가 되길래 저도 방향을 틀었습니다.

## 공식 문서 링크 수정 PR 작성

Spring Security 6 버전대가 되면서 Spring Framework 도 6 버전이 최소라 Tomcat 과 Netty 구버전이 지원되지 않는데, 링크가 구버전 링크로 되어있어 수정한 PR([링크](https://github.com/spring-projects/spring-security/pull/13565))을 올렸습니다.

하지만 역시나 무응답...

## Issue 부터 올리는 걸로 방식 변경

역시나 응답이 없길래 Issue를 안올리면 안봐주나 싶어서 이번에는 더 사소하게, 공식 문서의 링크 1개를 수정 요청하는 Issue([링크](https://github.com/spring-projects/spring-security/issues/13577))를 먼저 올렸습니다.

Issue 를 올렸더니 직접 PR 을 올려줄 수 있겠냐는 응답이 왔습니다. 감격...!

![issue에 대한 응답](/assets/img/2023-08-09-first-contribution-to-spring-security/01-issue.png)

감격 받아서 빠르게 PR([링크](https://github.com/spring-projects/spring-security/pull/13593))을 작성했습니다. 아직 merge 되지 않은 한 친구가 이 친구입니다.

이때 까지만 해도 사소한건 issue 안올려도 된다더니 거짓말이라 생각하고 있었습니다.

## 뒤늦게 온 merge 소식

잊고있던 이전에 제출했던 PR 이 merge 됐다는 소식이 갑자기 들려왔습니다.

![코드 가독성 개선 PR이 merge 됐음을 알리는 email](/assets/img/2023-08-09-first-contribution-to-spring-security/02-mail.png)

별거는 아니지만, 다른 사람들이 사용하는 오픈소스에 제 코드가 반영됐다는게 너무 기분이 좋았습니다. 그리고 오늘 아침에는 공식 문서 링크 수정한 것도 merge 됐다는 소식까지 받았습니다.

![링크 수정 PR이 merge 됐음을 알리는 email](/assets/img/2023-08-09-first-contribution-to-spring-security/03-mail2.png)

## Outro

오픈소스에 기여하면서 Git, Github 도 배우고, Java의 몰랐던 기능도 배울 수 있는 좋은 기회였던 것 같습니다. 그런데 지금 다시 보니 PR 올릴 때, 왜 이렇게 수정해야되는지 자세히 적지 않아 조금 아쉬운 것 같습니다. 사소한 것이다 보니 이번에는 merge가 되긴 했지만, 다음에는 왜 이렇게 변경해야 되는지, 그리고 변경을 통해 무엇을 얻을 수 있는지 PR 을 작성하면서 같이 작성하는 것도 좋은 연습이 될 것 같습니다.

1개 남은 PR도 나중에는 merge가 되지 않을까 생각됩니다. 오픈소스를 관리하는 곳 마다 다르긴 하겠지만, Spring Security 에 기여하고 싶으신 분들은 느긋함을 갖고 기다려보시는게 좋겠습니다.
