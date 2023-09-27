---
layout: post
title: Flashcards (1) 도메인 모델 초안 그리기
categories:
- Projects
- Flashcards
tags:
- Flashcards
date: 2023-09-26 12:59 +0900
---
## Intro

시간 제한을 걸어두지 않으니까 너무 잘 하려는 마음에 프로젝트를 시작조차 하지못하고 있습니다. 주제 선정부터 시간이 오래걸려서 Java 배우고나서 처음 Console로 만들었던 Flashcards 앱([링크](https://github.com/limvik/simple-flashcards))을 웹앱으로 다시 만들어보기로 결정했습니다.

먼저 요구사항을 생각나는대로 정리해본 후 도메인 모델 설계를 시작합니다.

## 요구사항

그냥 일단 Flashcards 에서 만들고 싶은거 생각나는대로 일단 적어봅니다.

| Type | Requirement | Plan |
|--|--|--|
| 기능 | 사용자 회원 가입 및 로그인 | ✅ |
| 기능 | 카드 덱 생성/조회/수정/삭제 | ✅ |
| 기능 | 카드 생성/조회/수정/삭제 | ✅ |
| 기능 | 학습 간격은 Spaced Repetition 알고리즘으로 자동 계산 | ✅ |
| 기능 | 카드 태깅 |  |
| 기능 | 태그 그룹화 |  |
| 기능 | 태그 입력 시 태그 추천 |  |
| 기능 | 카드 덱 별 기본 태그 설정 |  |
| 기능 | 스터디 그룹 |  |
| 기능 | 카드 공유 |  |
| 기능 | 카드 학습 일정 확인 |  |
| 기능 | 타이머 |  |
| 기능 | 원하는 카드만 복습 기능(예. 시험 전 표시해둔 카드만 복습) |  |
| 기능 | 카드와 관련된 자료 저장 및 링크 연결 |  |
| 기능 | 작성한 카드 내용을 기반으로한 요약 및 문제 생성 |  |
| 기능 | 구글 드라이브에 데이터 저장 - Bard로 요약, 검색 등 가능성 열어둠 |  |
| 기능 | 블로깅 기능 - 카드와 연결(구체화 필요) |  |
| 기능 | 자료에서 핵심을 요약하고 카드 문제로 자동 변환 |  |
| 기능 | 사용자 작성 답안에 대한 피드백 |  |
| 기능 | Anki Collections import 기능 |  |
| 비기능 | 모바일 앱 |  |
| 비기능 | 데스크톱 앱 |  |
| 비기능 | 브라우저 extension(웹 페이지에서 학습 내용 추출 목적) |  |
| 비기능 | Obsidian Plugin |  |

브레인스토밍 하듯 그냥 생각나는대로 다 적어보았습니다. 그리고 이번에 할 것만 Plan 열에 ✅ 표시해 봤습니다.

## 사용자 스토리(User Story)

[Designing APIs with Swagger and OpenAPI](https://www.manning.com/books/designing-apis-with-swagger-and-openapi) 에서 본 아래 양식을 바탕으로 사용자 스토리를 작성해 봅니다. 이전에 GPT로 사용자 스토리를 만들어서 동사와 명사를 추출해보기도 했었는데, 나름 사용되는 방법인가 봅니다.

> As a \<role> I can \<capability>, so that \<receive benefit>.  
> so that 절은 선택 사항이며, 사용자 스토리 배경 정보 제공  
> Given \<prerequisite>, I can \<capability>.

Flashcards 웹앱에서 생각나는 역할(role)은 학습자(Learner), 관리자(Admin) 두 가지 역할이 있습니다. 현재 진행할 기능을 바탕으로 역할별 사용자 스토리를 생각나는대로 쭉 작성해 보겠습니다.

### 학습자(Learner)

- 나는 새로운 계정을 만들 수 있고, 로그인 할 수 있다.
- 나는 내 계정 정보를 수정할 수 있다.
- 나는 카드 덱을 만들 수 있으며, 카드 덱 안에 하위 카드 덱을 생성할 수 있다.
- 나는 카드를 생성할 수 있으며, 카드는 원하는 카드 덱에 저장된다.
- 나는 내가 가진 카드 덱 목록을 조회할 수 있다.
- 나는 내가 가진 카드 목록을 조회할 수 있다.
- 나는 내가 가진 카드 덱 중 하나를 선택하여, 하위에 포함된 모든 카드로 학습을 시작할 수 있다.
- 나는 학습을 시작하면 카드 앞면을 보고 답을 입력한 후 뒷면에 작성된 답안과 비교할 수 있다.
- 카드를 뒤집어 답을 확인했다면, 알고리즘에 의해 계산된 다음 학습 일정을 난이도에 따라 선택할 수 있다.
- 생성된 카드 덱이 있다면, 카드를 생성할 수 있다.
- 생성된 카드 덱이 있다면, 학습자는 카드 덱을 삭제함으로써 카드 덱에 포함된 카드와 함께 동시에 삭제할 수 있다.
- 생성된 카드가 있다면, 학습자는 카드를 통해 학습할 수 있다.
- 생성된 카드가 있다면, 학습자는 선택한 카드를 삭제할 수 있다.
- 생성된 카드와 카드 덱이 있다면, 학습자는 카드와 카드 덱을 수정할 수 있다.

### 관리자(Admin)

- 나는 로그인할 수 있고, 관리자 기능에 접근할 수 있다.
- 나는 내 계정 정보를 수정할 수 있다.
- 나는 학습자 계정을 수정할 수 있다.
- 나는 학습자 계정을 삭제할 수 있다.
- 나는 학습자가 생성한 카드 덱과 카드를 수정할 수 있다.
- 나는 학습자가 생성한 카드 덱과 카드를 삭제할 수 있다.
- 나는 학습자가 생성한 카드 덱과 카드를 조회할 수 있다.

요구사항과 사용자 스토리를 바탕으로 명사에서 객체를, 동사에서 행동을 추출해 보겠습니다.

### 명사

객체를 식별하기 위해 명사를 뽑아보자면, 아래와 같습니다.

- 학습자
- 관리자
- 계정
- 카드 덱
- 카드
- 학습 일정
- 알고리즘

학습자, 관리자, 계정은 `사용자(User)` 객체로 통합하고, 카드 `덱(Deck)`, `카드(Card)` 는 각각의 객체로 볼 수 있습니다. `학습 일정(Plan)` 의 경우 카드와 관련된 내용이므로 카드에 속한 정보 객체가 됩니다.

### 동사

각 객체의 행동이 될 수 있는 동사를 찾아보면 아래와 같습니다.

- 조회하다
- 생성하다
- 수정하다
- 삭제하다
- 로그인하다
- 학습하다
- 뒤집다
- 계산하다
- 계획하다

다음 학습 일정을 선택한다는 표현에서 선택한다는 조회하다와 혼동이 있을 수 있어 어떻게 표현할까 고민하다가, 언제 공부할지 계획하는 행위이므로 `계획하다` 를 추가하였습니다.

## 도메인 모델

앞서 사용자 스토리에서 추출한 객체와 동사를 바탕으로 도메인 모델의 다이어그램을 간단하게 그려보면 아래와 같습니다.

![도메인 모델 초안](/assets/img/2023-09-26-draft-flashcards-domain-model/01.domain-model.png)

위 그림에서 알고리즘(Algorithm)과 카드(Card) 객체는 Anki에서도 사용하는 [SM-2](https://supermemo.guru/wiki/SuperMemo_1.0_for_DOS_%281987%29#Algorithm_SM-2) 알고리즘을 기준으로  작성했습니다. 구현 시에는 알고리즘 교체 가능성도 고려해서 구현을 해야겠습니다.

## Outro

드디어 스타트를 끊었습니다. 오래 붙들고 있어봤자 답이 안나오는데, 뭐 이리 쓸데없는 고민만 많이 한건지 스스로에게 답답함을 느낍니다.

다음은 OpenAPI 문서 작성을 진행해보겠습니다.