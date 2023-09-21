---
layout: post
title: Manning Liveproject 사용 후기
categories:
- etc
date: 2023-09-17 13:14 +0900
---
## Intro

뭔가 만들어야지 하면서 시간이 많이 지났습니다. 가장 큰 이유는 자기소개서 쓰느라 많은 시간을 보냈기 때문이고, 다른 하나는 Manning 출판사의 Liveproject 를 하고 있었기 때문입니다.

사용하려던 기술과 딱 맞아 떨어지는 REST Services with Spring and JPA([링크](https://www.manning.com/liveprojectseries/design-implement-rest-services-spring-jpa)) 라는 주제의 Liveproject가 있어 구매해서 사용해봤습니다. 사용을 고려하시는 분을 위해 후기를 공유합니다. 기술적인 내용은 없습니다.

## Manning

Manning 출판사를 잘 모르시는 분을 위해 북 커버를 보여드리면, '아 그 출판사' 하게 되실 것 같아서 북 커버 하나를 샘플로 올립니다.

![Manning 출판사의 북 커버](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/01.manning-book-cover.png)

이 출판사 책이 번역돼서 나오면 가끔은 한국화하지 않고, 원서 북 커버 그대로 사용해서 이러한 스타일의 책을 한 번쯤은 보셨을 거라 생각됩니다.

## Price

Liveproject의 가격은 보통 40 달러 대입니다. 20 달러 대의 가격도 있지만, 대체로 40 달러 대입니다.

![Live projects 목록](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/02.price-liveprojects.png)

여러 개의 project로 분할되어있어 개별 project를 구매할 수 있지만, 하나 당 19.99 달러이고 보통 6개로 구성되어 있는데, 연속되는 project라 그냥 한꺼번에 구매하는게 낫습니다.

![개별 project 목록](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/03.price-each-project.png)

하지만 저는 돈이 많이 없는 관계로 더 저렴하게 구매하려고 `subscription`을 구매했습니다.

![subscription 가격표](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/04.price-subscription.png)

월 24.99 달러로 Live project 전체를 구매할 수 있고, 구독 기간 동안에는 온라인으로 Manning 출판사의 모든 콘텐츠를 이용할 수 있습니다.

Liveproject 만 구매하면 추가적인 자료가 필요한 경우가 많다보니, 가격도 저렴하고 더 많은 콘텐츠를 이용할 수 있는 Subscription 이 더 효율적입니다.

Liveproject를 접근하려면 pro를 구매해야하고, subscription 목록에 써있듯 한 사람은 추가로 이용할 수 있으니, 같이 반반 나눠서 내면 더 저렴하게 이용할 수 있습니다.

## Liveproject

Liveproject가 어떻게 진행되는지 간단하게 소개 드리겠습니다.

Liveproject는 Manning 출판사에서 온라인으로 멘토가 작성한 커리큘럼에 따라 프로젝트를 진행합니다.

90일간 project에 포함된 책을 전체 열람할 수 있습니다.

![book resources](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/05.book-resources.png)

필요한 내용이 있는 책도 상황에 따라 추가로 알려줍니다.

![추가적인 book resources](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/06.additional-book-resources.png)

전체적인 흐름은 커리큘럼에 따라 Workflow가 주어지고, 무엇을 해야하는지 알려줍니다. 어려우면 참고하라고 도움말이 있기도 합니다.

![workflow](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/07.workflow.png)

너무 어려우면 멘토에게 질문할 수도 있습니다. 멘토 분이 외국사람이다 보니 물리적인 한계가 있어 바로바로 답변이 달리지 않는 것은 감안해야 합니다.

![멘토에 질문한 화면](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/08.discussion.png)

저는 도커 초보라 container name을 잘못 설정해서 헤메이다가 질문해봤었습니다. 멘토에게 공개되지 않은 장소에서 메시지를 보낼 수도 있는데, 저는 나중에야 알았습니다.

그리고 단계별 project를 완료하면, 개별 GitHub 저장소에 커밋한 후 링크를 제출합니다. 참고로 과제용 Github 저장소 생성 요청을 하면 Manning 계정에 private 저장소가 생성된 후 초대 요청이 옵니다.

![제출화면](/assets/img/2023-09-17-review-manning-liveproject-rest-api-with-spring/09.submit.png)

제출하고나면 solution을 볼 수 있고, 제출 전에 solution 열람을 요청하면 Certification을 받을 수 없으니 주의해야 합니다. 그런데 Certification 받으려면 15분 정도 멘토랑 면담해야 한다는데... 스피킹 연습을 해야 하나...

Liveproject는 이런식으로 관심있는 주제를 학습해볼 수 있고, Subscription 이용 시 Manning 출판사의 모든 책에 접근할 수 있어 한국 교육 업체들에 비해 상당히 저렴한 가격으로 학습이 가능합니다.

## REST Services with Spring and JPA

간단하게 왜 선택했고, 현재 잠시 중단했는데 왜 중단했는지 등을 적어보겠습니다.

### 선택한 이유

이전에 겪었던 문제점을 바탕으로 REST API를 개선해서 만들어보려고 했는데, 제가 원하던 것을 다 제시하고 있었습니다.

>skills learned  
> Make an application with `Spring Boot`, expose REST services with Spring Web, generate a SwaggerUI and `OpenAPI docs` with Springdoc, access a database with `Spring JPA`, manage database versions with `Liquibase`, Secure a `REST API` with `Spring Security` and `OAuth 2`, run an application in `Docker Containers`

이전에 Liquibase 말고 FlyWay를 사용해보는 것으로 결정하는 글을 작성([링크](/posts/liquibase-vs-flyway-vs-mybatis-migrations/))하기도 했지만, Liquibase 가 진짜 어려운지 사용해보는 것도 좋을 것 같았습니다. (결론적으로 project에서 사용한 수준으로는 Liquibase 도 크게 어려운점은 없었습니다.)

제가 주력으로하는 Spring, 그리고 이외에도 제가 관심있어하고, 사용해봤지만 잘 사용하지 못해 개선해보고 싶었던 기술들을 다루고 있어 선택하게 되었습니다.

### 기대했지만 얻지 못한 것들

#### 1. 빠른 지식 습득

제가 책 대신 동영상이라던가 이런 Liveproject 같은 것을 구매할 때는 필요한 지식만 `빠르게` 익히는 것을 기대하고 구매하게 됩니다. 각 기술의 문서를 읽어보는게 가장 좋은 방법이지만, 시간이 상대적으로 오래걸린다는 단점이 있습니다. 현업에서는 어떻게 사용하는지 등 최소한 기술을 잘못 사용하지는 않게 해줄 것을 기대합니다.

하지만 최소한의 가이드로 직접 기술 사용법을 찾아서 적용해보고, 멘토의 Solution과 비교해보는 방식이라 시간 절약은 기대하기 어려웠습니다. 그래도 멘토에게 질문할 수 있으니 그냥 책만 보는 것 보다는 훨씬 낫습니다.

#### 2. 멘토의 노하우

책을 포함해서 어떤 기술적인 자료를 돈 주고 구매할 때는 항상 기대합니다. `어떻게` 기술을 사용했는지도 궁금하지만, 다양한 구현 방법이 있는데 `왜` 그렇게 구현했는지, 어떤 장점이나 단점이 있는지 등 일반적으로는 구하기 어려운 지식을 기대합니다.

하지만 Solution은 단순히 코드만 제공되었습니다. 물론 멘토에게 질문하면 되지만, 질문을 위해서는 어느정도 제 생각이 필요한데, 제 생각을 정리하기 위한 지식들을 얻기 위해 얼마만큼의 시간이 필요할지 가늠이 안돼서 취업준비하고 있는 상황에 개인 흥미를 위한 공부에 시간 쓰기가 어려웠습니다. (제가 꼰대라 단순한 '왜 이렇게 했어요?' 라는 질문은 답변해주는 사람에게 답변 범위를 좁힐만한 정보를 제공하지 않아 무례하다고 생각해서 그런걸지도...?)

#### 3. API 설계 과정

OpenAPI docs를 언급하고 있어서, API 설계 노하우도 배울 수 있을 것이라 기대했지만, API 설계보다는 기술의 사용법에 초점이 맞춰져 있었습니다.

### 기대 밖으로 좋았던 점

#### 1. 몰랐던 것을 많이 알게 됨

Spring 컨테이너 이미지를 만들 때 Layered-jar로 만든다던가, GitHub Packages 가 컨테이너 이미지 저장소라는 것 등 이외에도 관심을 두지 않거나 몰랐던 사실을 많이 알게되었습니다.

#### 2. 매번 미루고 있던 것을 억지로라도 하게 됨

Spring Data JPA 를 간단하게만 사용해봐서, 1:다 관계만 사용해봤었는데 커리큘럼에 제시된 관계가 1:1, 1:다, 다:다 관계를 모두 다루고 있어 한 번씩 사용해볼 수 있었습니다.

또 DTO pattern([링크](https://www.baeldung.com/java-dto-pattern))을 사용해 보려다가, 생각만하고 실천은 안하다가 과제 제출하려고 만들어보게 됐습니다.

### 중단한 이유

많은걸 배워서 좋았지만, 많은걸 배워서 중단합니다.

뭔가 말이 앞뒤가 안맞는 것 같지만, 많은 내용을 제가 다 바로 소화하지 못해서 중단한다는 표현이 정확하겠습니다. 더 깊이 들어가보자면 취업 준비 때문에 마음이 급한데, 취업과는 조금 먼 거리에 있는 지식을 천천히 책 읽으면서 학습할 마음의 여유가 없는 것 같습니다.

아직 진행하지 않은 project에도 처음 보는 것들이 많이 남아있습니다. 멘토의 Solution에 대해 나의 생각과 함께 왜 그렇게 Solution이 작성되었는지 공부하면서 물어볼 수 있을 때 즈음 다시 진행해볼 예정입니다.

## Outro

지금 마음속 조급함만 없으면, 저렴한 가격에 양질의 학습자료를 마음껏 활용할 기회가 될 수 있을 것 같습니다. 하지만 취업 준비자에게는 코딩테스트가 더 중요하고, 뭔가 가시적인 것을 보여줘야 할 것 같은 마음이 들어 불안불안합니다.

그리고 다른 Liveproject는 멘토에 따라 달라질 수 있으므로, 모든 Liveproject가 이렇다고 일반화할 수는 없습니다. Liveproject 구매 전에 Manning에 문의하는 등 자신이 얻고자 하는 걸 얻을 수 있는지 알아보는게 좋겠습니다. Subscription 구매해서 살펴보는 게 가장 좋기는 할 것 같습니다.

Liveproject 구매를 고려하시는 분에게 도움이되었기를 바랍니다.
