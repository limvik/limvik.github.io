---
layout: post
title: 간단한 AWS CodeWhisperer 사용기
categories:
- Tools
tags:
- AWS
- CodeWhisperer
- Codegen
date: 2023-04-17 22:54 +0900
---
간단한 AWS Codewhisper 사용기

## Intro

AWS CodeWhisperer는 AWS에 가입하지 않고도 이메일 등록만하면 `개인은 무료`로 Github copilot 같은 코드 자동 생성기를 쓸 수 있다고 해서 Java로 sqlite 연결하는 클래스와 테스트를 만들면서 사용해봤습니다. 다른 것들을 써본적이 없어서 그런지 비교는 못하고 초보자로서 굉장히 만족스럽습니다.

## AWS CodeWhisperer 설치

워낙 간단해서 굳이 여기에 다시 정리할 필요는 없을 것 같습니다. 문서([링크](https://docs.aws.amazon.com/codewhisperer/latest/userguide/whisper-setup-indv-devs.html))도 굉장히 짧게 안내합니다. 그리고 영상도 아니고 아래와 같이 gif 로 설치방법을 보여줍니다.
![Amazon CodeWhisperer 설치 과정](https://docs.aws.amazon.com/images/codewhisperer/latest/userguide/images/cwspr-in-2-min_1x.gif)

뭔가 id를 만들기는 해야되기는 한데, AWS Builder ID 생성이니까 AWS 가입 안해도 된다는 말이 사실이기는 하네요.ㅎㅎ

## 짧은 사용기

초보자 티 내면서 처음에는 주석을 써야 코드가 나오는 줄 알았는데, 코드를 쓰다보면 다음에 쓸 코드를 추천해줬습니다.

![Singleton getInstance method 자동 생성](/assets/img/2023-04-17-aws-codewhisperer-usage/2023-04-17-codewhisperer-example1-singleton.png)

`tab` 키를 누르면 바로 적용이 되고, 마음에 안들면 `좌우 화살표` 키를 통해 여러 추천 코드를 확인해 볼 수도 있습니다.

Test에 사용할 메시지도 추천해줘서 좋았습니다.

![Test message 추천](/assets/img/2023-04-17-aws-codewhisperer-usage/2023-04-17-codewhisperer-example2-message.png)

아래처럼 boolean 에 assertNotNull 을 추천해주는 허접함도 보여주지만 이정도는 충분히 넘어가줄만 합니다.

![Test Code 생성](/assets/img/2023-04-17-aws-codewhisperer-usage/2023-04-17-codewhisperer-example3-next-test-logic.png)

다른 해볼만한 test를 발견하는데도 쓸 수 있지 않을까 싶어서 test 만 입력해보니 다른 test를 알려줍니다. 뭐 대단한 로직을 테스트 하는 것은 아니라, 다른 클래스의 test 를 더 만들어보면서 뭘 추천해주는지 봐야겠습니다.

![Test 추천](/assets/img/2023-04-17-aws-codewhisperer-usage/2023-04-17-codewhisperer-example4-new-test.png)

NotNull 이 아닌걸 봐서 그런지 이번엔 정상적으로 assertTrue로 추천을 해줬습니다.

![Test Code 생성](/assets/img/2023-04-17-aws-codewhisperer-usage/2023-04-17-codewhisperer-example5-new-test-logic.png)

## Outro

코드 자동생성기 사용도 처음이고 개발도 초보라 평가는 어렵지만, 쳐야 할 코드가 미리 제시되니 정말 편리합니다. 비용이 부담돼서 Github copilot 같은 것을 쓰기 주저했던 분들에게도 좋을 것 같습니다. 설치도 간단하니 꼭 한번 써보시는걸 추천드립니다.
