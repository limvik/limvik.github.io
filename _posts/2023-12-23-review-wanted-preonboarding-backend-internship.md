---
layout: post
title: 원티드 프리온보딩 백엔드 인턴십 후기
categories:
- etc
date: 2023-12-23 12:53 +0900
---
## Intro

원티드 프리온보딩 백엔드 인턴십을 드디어 수료 완료했습니다.

![수료 완료 메시지](/assets/img/2023-12-23-review-wanted-preonboarding-backend-internship/01.celebrating-message.png)

진짜 인턴십은 아니고, 교육입니다.

수료 완료 결과는 20일에 받았고, 원래 바로 후기 글을 써보려고 했는데 계획대로 흘러가지 않다보니 부정적인 감정이 가득해져서 쓸모없는 감정 배설 글이 될 것 같아 조금 미뤘습니다.

부정적인 감정 좀 가라앉히고 생각해보니, 목표했던 기간 내 취업이라는 성과를 얻지는 못했지만, 저의 문제점을 살펴볼 수 있는 기회가 됐고, 소통이 잘 되는 팀원들을 만나 소통의 중요성을 다시 한 번 느껴볼 수 있는 기회였습니다.

어떻게 진행됐는지 차례로 살펴보면서 기억을 되짚어 보겠습니다. 다음 차수 참여를 준비하고 계신 분들께 도움이 됐으면 합니다.

## 기본 정보

원티드 프리온보딩 백엔드 인턴십에 대해 전혀 모르는 분도 있을 수 있으니 간단하게 설명 드리겠습니다.

원티드 접속하셨던 분들은 챌린지는 매달 하다보니 가끔 보셨을 것 같은데, `챌린지`는 2주 동안 하나의 주제를 잡고 주 2회씩 지식 전달을 하는 것이 목적이라면, `인턴십`은 1개월 동안 프로젝트 수행 및 라이브 강의 수강하고, 2~3주 동안은 이력서 특강 및 실제 지원으로 이루어집니다.

그리고 인턴십은 인원 제한(100명)이 있어서 `사전 과제`를 바탕으로 인원을 `선발`합니다. 제가 지원할 때는 경쟁률이 대략 3:1 이라고 했습니다.

자세한 정보는 원티드([프리온보딩 링크](https://www.wanted.co.kr/events/pre_ob_be_7))에서 확인해 보실 수 있습니다.

이제 각 과정에 대한 썰을 풀어보겠습니다.

## 사전 과제

인원 선발을 위해 각자 주력 언어를 선택해서 `사전 과제`를 수행하고, `GitHub 저장소 링크`와 참고용 `이력서`를 제출합니다.

`이력서`는 선발에 아무 상관 없다고 합니다. 그런데 이력서 교육 때 강사 분이 일부분 피드백 해주시기도 하니까 그래도 진지하게 써보는게 좋은 것 같습니다.

`사전 과제`는 당시에 Spring 에서 슬라이스 테스트(Slice Test) 작성하는 방법을 책에서 막 봤던 터라 떨어지더라도 이것만은 연습해 봐야겠다 싶어서 Controller, Service, Repository 계층 별로 테스트 작성을 열심히 했었습니다.

### 사전 과제 팁

`사전 과제는 빨리 제출하는게 장땡`이라는 경험을 이전 프리온보딩 인턴십 지원 때 했기 때문에, 테스트 작성 연습해보는 목적을 달성한 후, 기본 요구사항과 추가 점수 항목은 점수 얻을 정도만 만들고, 가산점 받을 수 있는 기간에 제출하였습니다. 혹시 보실 분들은 [GitHub 링크](https://github.com/limvik/wanted-pre-onboarding-backend)에서 제가 수행한 사전 과제를 보실 수 있습니다. (빨리 냈을 때 이정도면 통과 가능하다!)

사전 과제는 인턴십 참여가 목적이므로 퀄리티는 조금 포기하더라도 `시간 가산점` 받는게 가장 좋은 선택입니다. 물론 기본적인 요구사항과 추가 가산점 항목들은 완성해야한다는 전제가 있기는 합니다.

### 선발 이후 - 결제

사전 과제를 통과하여 선발되면, 보증금(?)과 같은 참가비 30만원을 결제 합니다.

![02. 참가비 결제 화면](/assets/img/2023-12-23-review-wanted-preonboarding-backend-internship/02.join-fee.png)

팀 프로젝트 탈주 방지 및 열심히 참여할 동기를 만들어 주기 위한 목적이라고 합니다. 그리고 수료 조건 달성하면 환불해 줍니다.

## 프로젝트

### 프로젝트 인원

조 마다 조금씩 다르기는 했지만 저희 조는 4명이었고, 5명인 조까지 봤습니다.

### 프로젝트 기간

프로젝트는 `팀 프로젝트 2회, 개인 프로젝트 2회`를 수행합니다. `1개월` 동안 수행하다보니 각 프로젝트에 주어진 시간은 `1주일`이라 시간은 촉박한 편입니다.

### 프로젝트 진행

#### 요구사항

프로젝트 요구사항을 강사님이 예고된 시간에 공지하면 팀원과 살펴본 후, 강의 시간에 강사님이 요구사항을 설명하고 질문받는 식으로 진행했습니다.

각 프로젝트의 구체적인 요구사항은 강사님 저작물이라 넣지 않겠습니다.

#### 프로젝트 구조 협의

첫 팀 프로젝트 전에 팀 프로젝트에서 재활용하면서 사용할 프로젝트 구조를 협의합니다. Java/Spring의 Package 구조를 협의했는데, 각자 선호하는 Package 구조를 보고 배울 수 있어서 흥미로운 시간이었습니다.

#### 첫 번째 팀 프로젝트

첫 번째 팀 프로젝트는 소셜미디어 글에 대한 탐색 및 통계 정보를 제공하는 REST API를 주제로 만들었습니다([GitHub 링크](https://github.com/pre-onboarding/Social-media-integrated-feed)). 

##### 회원 기능 구현 담당

저는 회원 가입/로그인 및 JWT 발급/인증 기능을 맡아서 제작했습니다. 기존에 만들어놓은 코드를 재사용하려다가 Spring Security 의 [OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html) 라이브러리 코드를 일부 수정해서 사용해봤습니다. 제가 만들었던 인증 코드와 비교해보면서 학습해볼 기회가 되기도 했습니다.

##### 반성

여기서 잘못 생각했던게 OAuth 2.0 구현하는 것은 아니니까 Resource Server 라이브러리를 사용하지 못하겠다는 생각이었습니다. 최근에 [OAuth 2.0 구현하는 Spring Academy 강의](https://spring.academy/courses/spring-academy-secure-rest-api-oauth2)를 보니 Resource Server 라이브러리에서 Token 발급하는 옵션도 있는 것을 보고 역시 멀었구나 하는 생각을 했습니다. 단지 OAuth 2.0이 아닐 뿐 자원의 상태를 전송하는 API Server인데, 왜이리 선을 그으려 했는지 모르겠습니다.

그런데 첫 프로젝트에 짧은 기간이라 그런가 다들 어버버하다가 끝나버렸습니다. 다들 이번 프로젝트를 망했다고 생각하는게 통했는지, 문서화도 안하고 다음 프로젝트로 넘어갔습니다.

#### 두 번째 팀 프로젝트

두 번째 팀 프로젝트는 지역 기반 맛집 추천 REST API였습니다([GitHub 링크](https://github.com/pre-onboarding/yamyam)). 짧은 시간이지만 서비스 만드는 기분도 내볼겸 `yamyam` 이라는 이름도 지어줬습니다.

##### 소통 문제 개선

첫 번째 팀 프로젝트 시작 전에는 모두 소통의 문제를 느꼈는지, 이를 개선하기 위해 고민했습니다. 그래서 PR 양식도 여기저기 기업 기술 블로그를 보면서 새롭게 정하고, 우테코 프로젝트를 참고해서 GitHub Discussion 같은 것도 따라해봤습니다. 가장 중요했던 것은 `1일 1PR` 규칙이었던 것 같습니다.

1일 1PR을 규칙으로 정하고 나니, 1일 1PR이 되지 않더라도 서로 어디까지 했는지, 어떤 문제가 있는지 등에 대해 소통이 잘 되는 경험을 할 수 있었습니다. 이때서야 소통 도구의 문제가 아니라 `소통하고자 하는 의지`와 `소통 시간`이 필수적임을 깨달을 수 있었습니다.

팀원분의 추천으로 KPT 회고([링크](https://techblog.woowahan.com/2677/))라는 걸 해봤는데, 다들 소통이 잘 돼서 좋았다는 의견이 나와서 내가 소통 빌런이 아니었음에 안도의 한숨을 내쉬었습니다.

##### 개선에 적극적이지 못했던 아쉬움

전국 시군구 데이터 csv 파일 업로드 기능을 맡게 되었는데, 파일 업로드에 대한 어려움을 토로하는 글을 많이 봐았던 터라 시간을 넉넉하게 잡았지만 너무 간단해서 당황했습니다. 특히나 시군구 정보가 몇년에 한 번 바뀔까 말까한 정보(예를들면, 강원도 -> 강원특별자치도)이고, 비밀 데이터도 아니며, 텍스트 데이터라 용량도 작아 어떤 기술적 시도를 해볼만한게 없었습니다.

뭔가 좀 허무해서 아직 담당자가 없었던, 시군구 데이터를 Redis에 캐싱해서 응답 속도를 향상시키는 일을 맡았습니다. Redis를 사용해본 적이 없어서 팀원들한테 민폐되는건 아닌가 걱정했는데, Spring Data Redis를 사용하면 Redis 사용하는게 이렇게 쉬울 줄은 또 몰랐습니다.

시간이 남아서 사용해보지 않았던 Discord Webhook 으로 메시지 보내는 기능도 예상보다 간단하게 해결했습니다.

여기서 문제는? 딱히 더 잘 해보려는 생각을 안했다는 점입니다. 무슨 상상을 하며 갖다 붙여봐도 괜찮은데 안해봤습니다. `스스로 어떤 전제를 하고 이슈를 만들어보라`는 팁을 얻어서 다음 준비 중인 프로젝트에서 시도해볼 계획입니다.

##### 유연하지 못했던 생각

제 담당은 아니었지만, 공공 데이터 포털에서 제공하는 음식점 데이터 API를 연동하는 요구사항이 있었습니다. 그런데 기본 키(Primary Key)로 사용할만한 데이터가 없어서 어떤 데이터를 Key로 사용할 것인지에 대한 문제가 있었습니다. 특히, 사업자 번호가 없다보니, 유일값을 갖는 데이터가 없었습니다.

데이터를 살펴보니 기존 데이터가 변경되지 않고, 새로운 변경 사항이 새로운 데이터로 추가되는 방식으로 보였습니다. 누적으로 쌓이는 데이터라면 대리키(Surrogate Key)를 만들면 될 것 같아, 팀원 대신 담당 공무원분한테 데이터 문의를 보냈습니다. 답변을 새로운 변경 사항만 누적적으로 추가되는게 맞다고 받기는 했는데! 아쉽게도 거의 일주일이 다 지난 상태라 이미 기능을 담당했던 팀원이 나름대로 복합키를 만들어 구현한 이후였습니다.

답변에는 원천 데이터를 제공하는 곳의 주소([`localdata.go.kr`](https://localdata.go.kr))가 첨부되어 있었고, 해당 API는 기존 데이터와 변동분을 제공하는 API를 별도로 제공하여 데이터를 쉽게 파악할 수 있었습니다. 요구사항에서 제시한 API만 고집하려다가 더 나은 데이터가 있는 API를 찾아볼 생각도 못했던 것 같습니다. 답변만 기다리면서 더 괜찮은 API가 있는지 직접 찾아볼 생각은 못했던게 조금 아쉬웠습니다.

#### 첫 번째 개인 프로젝트

첫 번째 개인 프로젝트는 사용자의 예산 관리 서비스 REST API 였습니다([GitHub 링크](https://github.com/limvik/budget-management-service)). 이 프로젝트도 서비스 만드는 기분 낼 겸, 개인화된 서비스를 강조하기 위해 `EconoMe`로 이름을 지었습니다.

##### 팀 프로젝트 보다 많은 요구사항

개인 프로젝트라도 팀 프로젝트 하던 팀원들과 같이 모여서 요구사항에 대해 이야기하는 시간을 가졌었습니다.

공통된 의견은 `이럴거면 팀 프로젝트로 이걸 내주지...` 였습니다. 팀 프로젝트를 금방 해버리는 팀이 많다보니까 아무래도 강사님이 조정을 하셨던거 같습니다.

##### 새로운 시도

이전 프로젝트에서 팀원들한테 민폐 끼칠까봐 너무 소심하게 프로젝트를 한 것 같아, 개인 프로젝트에서는 시간 안에 해볼만한 새로운 시도를 여러가지로 해봤습니다.

- `프로젝트 일정 관리`: GitHub Projects 를 이용해서 작업 일정 관리하고 시간을 배정하는 등의 작업을 해봤습니다. 생각보다 시간 예측을 엄청나게 못한다는 사실을 깨닫고 고려하지 못하는 점들이 무엇이 있나 고민해보면서 점점 예측 시간과 실제 시간의 간격이 줄어드는 것을 볼 수 있었습니다.
- `문서화 도구 변경`: 이전에는 OpenAPI Specification 문서와 Swagger 도구로 API 문서를 작성하다가 Spring REST Docs도 써봤는데, 자동으로 전체적인 문서가 만들어지는 것은 아니라서 개인적으로는 별로였습니다.
- `배포`: Jenkins 서버 별도로 운영하기에는 부담이 있어서 GitHub에서 제공하는 GitHub Actions 자료([링크](https://resources.github.com/learn/pathways/automation/essentials/automated-application-deployment-with-github-actions-and-pages/))를 보면서 GitHub Container Registry에 컨테이너를 올리고 AWS EC2에 배포해보았습니다.
- `테스트 지표 개선`: 테스트 속도 개선도 해보고, 테스트 커버리지도 개선해보고 하면서 지표를 바탕으로 개선하는게 저랑 잘 맞는 것 같아서 재밌었습니다.
- `기타`: 지금은 책 읽으면서 실습용으로 사용하고 있습니다. 특히 JPA 책 읽으면서 몰랐던 기능 만날 때 마다 적용해보는 재미가 쏠쏠합니다.

#### 두 번째 개인  프로젝트

두 번째는 협업 도구 API 였던걸로 기억하는데 안했습니다. 당장 새로운 프로젝트를 한다고 해도 개선해서 적용해볼 지식이 없었고, 기능 구현도 딱히 난이도가 있는 것은 아니어서 요구사항만 많은 일주일짜리 프로젝트를 한다고 해도 크게 얻을 것이 없다고 판단했습니다.

강사님이 마지막 개인 프로젝트는 수료 조건에 안넣겠다고 하셔서 안하는걸로 쉽게 결정할 수 있었습니다.

#### 다음 프로젝트 계획

두 번째 개인 프로젝트를 할 시간에 앞서 첫 번째 프로젝트에서 언급한 새로운 시도들 중 일부를 해보면서 기존의 잘못된 것을 개선하는게 더 의미있게 느껴졌습니다. 그래서 다음 프로젝트는 만들어보고 싶은 서비스를 만들어보면서 장시간 개선해 나가는 경험을 해보려고 합니다. 이제는 연습용이 아니라 돈이 될만한, 사람들이 쓸만한 서비스를 만들어보고 싶기도 합니다.

## 강의

프로젝트 기간 동안에는 기술에 관련된 강의가 주 2회(화, 목) 3시간 동안 이루어지고, 이후에는 이력서 강의가 정해진 시간표에 따라서 유동적으로 진행되었습니다.

### 기술 강의

새로운 프로젝트 요구사항이 나왔을 때는 요구사항 설명하는데 주로 시간이 많이 사용됐고, 커리큘럼 상에 나온 내용은 다루어지지 않는게 많았습니다. 이 글 쓴다고 커리큘럼 다시 살펴봤는데, 단어조차 나오지 않은 것들이 꽤 많습니다. 

다룬 것들도 아무래도 양이 많다보니 시간 한계상 모두 다룰 수 없어, 적당히 겉핥기로 언급하는 정도라 그냥 커리큘럼 상에 나온거 직접 검색해서 보는 것과 큰 차이는 없었습니다. 강사님도 세부적인 내용은 알아서 찾아보라고 하셔서, 특별히 언급할 내용이 없는 것 같습니다. 그래도 강사님이 나름 열심히 질문도 받아주시면서 시간 내에서는 정보를 많이 제공하려고 노력하셨습니다.

### 이력서 강의

이력서 강의는 보통의 유튜브에 올라와 있는 내용과 비슷하지만, 조금 더 정돈된 느낌입니다. 강사님 혼자서 100명이 되는 인원을 다 봐주셔야돼서 많은 부분을 봐주시진 못하지만 한명 한명 자기소개서나 이력서 일부분 코칭해주셔서 꽤나 도움이 됐습니다.

## 이력서 작성 및 지원

다음은 저를 부정적인 감정에 휩싸이게 했었던 지원하기 단계 입니다.

### General 한 이력서 던지기 + 가볼만한 곳은 신경써서 지원하기

기한 내에 총 `20개의 기업을 원티드에서 지원`해야 수료가 가능합니다. 보통 기업 하나 지원하는데 2~3일 정도 알아보고 지원하는 저로써는 최대 60일이 걸리는 작업이라 다른 방법이 필요했습니다.

좋은 방법은 아니지만, General한 이력서를 작성해서 어차피 탈락할만한 기업(마음 편히 탈락시켜 주십쇼)과 어느 정도 가보고 싶은 기업(가면 좋고 아님 말고)을 적당히 섞어서 지원했습니다.

그리고 예상대로 연전 연패 중입니다. 자업자득에 예상했던 결과라 하더라도 나약한 인간으로써 거부당한 경험은 부정적인 감정에 휩싸이게 만들었습니다.

### 다른 채용 사이트에 지원해보고 해소된 부정적인 감정

뭔가 경험해보지 못한 상황들을 마주하니 원티드 플랫폼 자체에 대한 부정적인 감정이 생겨서 혹시나 해서 사람인으로 넘어가서 지원해 봤는데, 여기는 `이력서 열람만 해줘도 감사`한 수준이었습니다. 그제서야 요즘 채용 시장이 이렇구나 하면서 부정적인 감정이 사라졌습니다.

### 상황별 감정의 변화

| 상황 | 솔직한 첫 감정 | 현재 감정 |
|--|--|--|
| 서류 불합격 | 아 뭐 예상은 했는데, 기분 더럽네 | 불합격을 알려주는 회사라니 감사. 크게 될 회사인듯 |
| 열람 후 응답 없음 | 아주 지원자를 개무시하는구나 | 열람이라도 하는 곳이네. 인사 업무라는게 머릿속에 있기는 한 곳이구나. |
| 감감 무소식 | 음...? | 회사 망했나...? |
| 이력서 열람 없이 불합격 | 이미 해탈한 상태에서 겪어서 별 생각 없음 | 다시 생각하니 무례하긴 하네 |

예전에는 기업을 자세히 알아보지도 않고 General한 이력서를 작성하는 사람들이 이해가 안됐는데, 이제는 이해가 됩니다. 채용에 진심인 기업들은 이런 이력서를 어이 없어 할거고, 지원자는 열람을 할지 안할지도 모르는 기업에 시간 쓰기가 두려운 상황입니다.

그런데 생각해보면, 원티드 플랫폼은 하나의 이력서로 여러 곳에 동시 지원하는 기능을 제공하고 있습니다. 아마도 북미나 유럽처럼 간단한 이력서로 면접여부를 결정하는 컨셉인거 같은데, 아직 한국은 받아들이기 힘든 문화인거 같습니다.

### 다시 준비하는 첫 단계로 돌아간다면 해보고싶은 지원 전략

취업 못한 자의 추측일 뿐이니 가볍게 보시면 됩니다.

#### 쥐뿔도 없을 때 일단 지원해보기 전략

다시 개발자를 준비하는 처음으로 돌아간다면, 첫 프로젝트가 끝나고 바로 지원해볼 것 같습니다. 열람도 안하는 곳이나 열람하고 응답 없는 등의 기업은 향후 지원 목록에서 걸러내고, 불합격 통보를 해주는 곳 위주로 6개월 후 즈음 다시 지원해볼 것 같습니다. 그럼 지원해볼만한 목록도 많이 줄어드니 집중해서 그 기업들에 맞춰 준비해서 효과도 더 좋을 것 같습니다.

#### 산업군 넓게 고려하기 전략

다른 방법은 언어도 배우기 전에 가고 싶은 산업군을 정하고, 산업군 내에 많이 사용하는 언어를 선택해서 배우겠습니다.

저는 사실 HFT(High Frequency Trading) 해보고 싶어서 개발을 배우기 시작했습니다. 그런데 현실적으로 HFT 회사에 취직하려면 몇년은 공부해야돼서 HFT에도 많이 사용되고, 국내 다른 일자리도 많다는 Java를 선택했습니다. (HFT는 취미로 하는걸로...)

여기서 잘못한 점은 금융권은 전혀 고려하지 않았다는 점 입니다. 산업군을 너무 좁게 본거죠. 엊그제부터 진심으로 지원할 겸 사람인도 살펴보니 트레이딩 솔루션을 증권회사에 납품하는 회사도 있다는걸 뒤늦게 발견했습니다. 알고리즘을 만드는 건 아니지만, 근처에라도 갈 수 있다는게 참 매력적인데 지원할 명분이 없습니다...만! 일단 찌르기 시전했습니다. 어차피 기간 지나면 지원할 수도 없을테니까요. 예전 같았으면 포기했을텐데, 원티드의 강제 지원 경험이 저를 바꿔놓았습니다.

## 기대했던 것과 기대와 달랐던 점

앞에서 쓰다가 글이 길어져서 언급 안하고 넘어갔던 내용 중에 교육 과정이 기대와 달랐던 점을 정리해 보겠습니다.

### 경험도 많고 기술적으로 많은 것을 아는 강사님이 가르칠 것이라는 기대

사전에 설명회를 할 때 nhn 에서 근무하시는 능력있는 강사분이라고해서 기술적으로 많이 배울 수 있을 것이라 기대하게 만들었는데, 프리랜서하시는 분이 강사분으로 오셨습니다.

프리랜서 분이 뭐 능력이 떨어진다거나 경험이 없거나 하는 것은 아니지만, 강사님이 말하길 자신은 비즈니스 쪽으로 강점이 있는 사람이라 기술적인 부분은 다른 곳에서 찾아보는게 좋다고 하셨습니다.

저도 비즈니스 쪽 좋아하긴 한데, 강사님 성향이 너무 비즈니스쪽이라 저랑은 잘 안맞았습니다.

### 기업이 참여해서 과제도 내고 수행한 과제도 검토할 것이라는 기대

이전에는 기업 이름이 붙어있는 과제가 있는 것으로 보아 기업이 참여해서 과제도 내고 괜찮으면 면접도 보고 했던 것 같습니다.

제가 참여했던 차수는 기업 과제는 없었고, 강사님이 기업 과제들을 바탕으로 요구사항을 만들어서 과제를 수행했습니다.

사실 기업 과제를 한다는 것 보다도 기업과 연계된다는게 큰 의미가 있는건데 아쉬웠습니다.

이것도 사전에 어떤 기업이 참여하는지 얼마나 참여하는지는 알 수가 없어서 참여하는 시기에 따라 운이 있어야 하는 것 같습니다. 설명회에서는 무조건 참여하는 기업이 있는 것처럼 이야기하기는 합니다.

### 팀원들은 모두 취업을 목표로 열정적으로 할 것이라는 기대

선발 과제가 시간을 하루 이상은 투자해야되는 과제라 이런 수고로움을 감수할 정도면 팀원들이 꽤나 열정을 갖고 할 줄 알았습니다. 이어서 팀원들이 어떻게 열정이 없었는지 언급을 해야겠지만, 사실 앞에서 말한 기업참여가 없는 걸 다들 눈치 챈 순간 열정이 없어지는게 이해가 되는 상황이라 탓을 하기도 애매한 것 같습니다.

### 원티드 뱃지 달면 기업들이 열람도 하고 면접 제안도 올 것이라는 기대

이력서 지원 기간에는 참여자들에게 원티드 뱃지를 달아줍니다. 그러면 기업들이 이력서 열람도 많이하고, 개중에는 면접제안도 하나쯤은 오지 않을까 기대를 하고 있었습니다. 그런데 열람하는 기업이 딱 1개 있었습니다. 요즘 경기가 안좋으니 열람할 생각이 없는(=적극적으로 채용할 생각 자체가 없는) 것 같습니다.

## Outro

기대와 달리 아쉬운 점이 많기는 했지만, 그래도 혼자 했으면 못얻을 경험과 정보들을 얻기도 했습니다. 아마 원티드 아니었으면 여태 이력서를 1개도 넣어보지 않았을 겁니다. 아직 부족하다는 생각에 지원조차 못하고 있으신 분들은 원티드 프리온보딩 인턴십 참여해서 강제로라도 지원해보는 연습 해보는걸 추천합니다.

 비관적인 뉴스들도 많이 보이는데, 지치지 않고 다음을 준비하는게 중요하겠습니다.

[‘법인 파산 비율’ 올해 역대 최고…중소기업 잔혹사 본격화](https://n.news.naver.com/mnews/article/366/0000957148)
