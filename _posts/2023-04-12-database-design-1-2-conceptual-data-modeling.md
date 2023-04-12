---
layout: post
title: 데이터베이스 설계 연습1-2 개념 데이터 모델 ERD 그려보기
categories:
- Database
- DataModeling
tags:
- Database
- DataModeling
date: 2023-04-12 23:41 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/database-design-1-1-data-modeling-research/)에서 요구 사항은 잠시 묻어두고 ERD(Entity-Relation Diagram) 부터 그려보기로 했습니다. 그런데 이제 막 읽기 시작한 『클린 코드』에서 코드는 요구 사항을 표현하는 언어라고 합니다. 코드의 중요성을 강조하기 위함인데, 저에겐 정확한 요구 사항의 중요성으로 다가옵니다.

그리고 새롭게 참고할만한 자료([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=276))를 찾았습니다. 같은 웹 사이트의 DA 가이드를 보면 많은 자료를 보실 수 있습니다. 링크는 데이터 모델링 개요인데, 여기에는 이런 글이 있습니다.

> 시스템 개발 실패의 주요 요인을 무엇이라 생각하는가? 그것은 처음의 요구 사항을 충족시키지 못하는 설계자의 무능력을 들 수 있다. 즉 적절한 시스템이 개발되기 위해서는 요구 사항을 완전하고 정확하게 식별해야 한다.

뼈가 가루가 되게 뚜드려 맞는 느낌...?

요구 사항을 다시 고민해봐야 하나 고민되지만 배운걸 적용하고 경험해보는 것에 가장 우선순위가 있으므로, 일단은 머릿속에 있는 흐릿한 요구사항을 바탕으로 개념 데이터 모델의 ERD를 간단하게 그려보았습니다.

## 여러가지 ERD 표기법

ERD 표기법도 종류가 여러가지가 있습니다. Wikipedia([링크](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)) 에 나와있는 목록만 해도 9가지나 됩니다.

Bachman notation, Barker's notation, EXPRESS, IDEF1X, § Crow's foot notation (also Martin notation), (min, max)-notation of Jean-Raymond Abrial in 1974, UML class diagrams, Merise, Object-role modeling

![ERD](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f1/ERD_Representation.svg/320px-ERD_Representation.svg.png)

정보처리기사에서 봤던 표기법은 `Chen`과 `Martin / IE / Crow's Foot` 입니다. 도형별 의미와 `까마귀 발(Crow's Foot)`이 의미하는 것을 외웠던 기억이 납니다.

일단은 가장 널리 사용된다고 배웠던 `까마귀 발` 표기법을 사용해 보겠습니다.

ERD 도구도 굉장히 많은데, 인파님의 블로그([링크](https://inpa.tistory.com/entry/DB-%F0%9F%93%9A-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%AA%A8%EB%8D%B8%EB%A7%81-1N-%EA%B4%80%EA%B3%84-%F0%9F%93%88-ERD-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8))에서 봤던 [ERD CLOUD](https://www.erdcloud.com/)가 무료에 디자인도 제일 마음에 들어서 이를 이용해서 그려보겠습니다.

## 개념 데이터 모델 ERD 그려보기

DA 가이드([링크](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?pageid=5&mod=document&uid=284))와 위에서 언급한 인파님의 블로그를 참조하여 개념 데이터 모델부터 작성해 보겠습니다.

인파님은 `Chen` 표기법을 이용해서 개념 데이터 모델을 작성했지만, ERD CLOUD에 `Chen` 표기법은 없어서 `까마귀 발` 표기법을 사용하였습니다.

### 엔터티 후보 검토

저는 어설프지만 도메인 모델을 그리면서 식별([링크](https://limvik.github.io/posts/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5%EA%B0%9C%EB%B0%9C-%EC%97%B0%EC%8A%B51-%EB%8F%84%EB%A9%94%EC%9D%B8-%EB%AA%A8%EB%8D%B8-%EA%B7%B8%EB%A6%AC%EA%B8%B0/))했던 객체가 저의 엔터티(Entity) 후보입니다.

![도메인 모델](/assets/img/2023-03-27-domain-model-for-flashcards.png)

### ERD 그리기

도메인 모델에서 식별된 객체 중 중간자 역할로 아무런 상태를 갖지 않는 필기구(Pen) 객체 그리고 중간에 새로 식별했던 객체인 현황판(Board) 객체를 제외하고, 사용자(User), 보관함(Deck), 카드(Card), 플래너(Planner)를 핵심 엔터티로 하여 아래와 같이 ERD를 작성하였습니다.

![개념 데이터 모델 ERD](/assets/img/2023-04-12-database-design-1-2-conceptual-data-modeling/2023-04-12-erd.png)

문제 풀때는 식별자 관계 잘못된거 고르는게 굉장히 쉬운 문제인데, 막상 제꺼는 관계선 한줄 긋는 것도 쉽지 않습니다. 처음에는 `플래너` 엔터티에 `보관함식별자`도 필요하니까 `보관함` 엔터티와 관계를 표시했었습니다. 그런데 생각해보니 `카드` 엔터티가 갖고 있는 하나의 정보로서 `보관함식별자`가 의미 있는거지 `플래너`와 `보관함`이 어떤 관계를 갖는 것은 아니어서 삭제했습니다.

`플래너` 엔터티에서 `사용자식별자`는 PK와 FK를 표시하려고 2개를 추가하였습니다.

## Outro

ERD를 그려놓고 보니 아직 혼란스러운게 많습니다.

1. 보관함은 디렉토리 구조로 만들려고 계획하고 있습니다. 그런데 여러 계층을 갖게 될 때 상위의 모든 보관함과 하위의 모든 보관함이 다대다 관계를 갖는건지 아니면, 바로 위아래 계층만 고려해서 일대다 관계가 되는건지 고민이 됩니다.
2. 카드에 대한 학습 기록을 카드를 삭제 한다고 해도 남겨 놓는게 맞는건지, 카드를 삭제하면 같이 삭제가 되어야 하는건지 혼란스럽습니다. 이건 요구 사항에 따라 달라져야 할 것 같은데, 또 요구 사항에서 걸립니다.
3. 트랜잭션 모델링은 어떻게 시작해야될지 감도 안잡힙니다. 트랜잭션 모델까지 다 고려한 결과물이 개념 데이터 모델의 ERD가 되어야 하는건가 싶기도 합니다.
4. 어디까지가 정확하게 개념 데이터 모델링이고, 어디부터가 논리 데이터 모델링의 시작이라고 봐야할지도 잘 모르겠습니다.

혼란의 연속이지만, 일단 참고하는 블로그 글을 따라 개념 모델링은 간단하게 마무리하고 이후에 고민하던 것을  수정하면서 논리 데이터 모델링을 해봐야겠습니다.

## 참고자료
- [https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=278](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?mod=document&uid=278)
- [https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?pageid=5&mod=document&uid=284](https://dataonair.or.kr/db-tech-reference/d-guide/da-guide/?pageid=5&mod=document&uid=284)
