---
layout: post
title: 비밀번호 최소 길이는 8글자, 최대 길이는?
categories:
- Security
tags:
- Security
date: 2023-10-04 14:42 +0900
---
## Intro

회원가입 시 비밀번호 길이를 설정할 때 `최소 길이`는 명확한 지침이 있습니다.

KISA의 [패스워드 선택 및 이용 안내서](https://www.kisa.or.kr/2060305/form?postSeq=14&lang_type=KO)를 보면, 두 종류 이상의 문자구성과 `8자리 이상`의 길이로 구성된 문자열(문자종류는 알파벳 대문자와 소문자, 특수문자, 숫자 4가지) 또는 `10자리 이상`의 길이로 구성된 문자열(숫자로만 구성할 경우 취약할 수 있음)을 안전한 패스워드로 언급하고 있습니다.

미국의 NIST 문서([sp800-63b](https://pages.nist.gov/800-63-3/sp800-63b.html#:~:text=5.1.1.1%20Memorized%20Secret%20Authenticators))에서도 `최소 8글자`를 언급하고 있습니다. 

> Memorized secrets SHALL be `at least 8 characters` in length if chosen by the subscriber.

마찬가지로 `최대 길이`는 명확하게 지침을 주고 있지 않고, Appendix의 Length 섹션에서 사용자가 원하는만큼 입력하게 하되, MB 크기의 길이를 허용하면 해싱하는데 시간이 오래 걸리므로 합리적인 선에서 제한을 하라고만 합니다.

> 글 업로드 후에 위 지침과 관련해서 흥미로운 글을 발견해서 글 하단에 추가로 업데이트 하였습니다.

찾아본 결과로는 비밀번호 최대 길이를 설정할 수 있는 마법의 공식은 없어서 아쉬웠지만, 최대 길이를 설정할 때 고려할만한 사항들을 생각해볼 수 있었습니다.

## 유명한 서비스들의 비밀번호 최대 길이

먼저 네이버부터 살펴보겠습니다.

### Naver

한국인에게 익숙한 네이버는 최소 8글자, 최대 16글자 입니다.

![네이버 회원가입 화면에서 비밀번호 초과 입력한 결과](/assets/img/2023-10-04-how-to-set-maximum-password-length/01.naver-password-length.png)

최대 길이가 생각보다 많이 짧습니다.

### Google

구글은 어떨까요?

최소는 8글자, 최대는 100글자입니다.

![구글 회원가입 화면에서 비밀번호 최소 길이 경고화면](/assets/img/2023-10-04-how-to-set-maximum-password-length/02.google-password-min-length.png)

![구글 회원가입 화면에서 비밀번호 최대 길이 경고화면](/assets/img/2023-10-04-how-to-set-maximum-password-length/03.google-password-max-length.png)

그런데 GCP 의 인증 서비스인 [Identity Platform](https://cloud.google.com/identity-platform/docs/password-policy?hl=ko)은 최대 비밀번호 길이를 `4096 글자`까지 설정 가능합니다.

다른 서비스를 근거로 비밀번호 최대 길이를 설정하기는 어려워보여서 추가적으로 더 찾아보지는 않았습니다. 간단하게 구글 검색 결과만 살펴보면 X(구 Twitter)는 써있는 곳마다 다르긴 한데, 100글자는 넘습니다.

### 네이버도 16글자인데 16글자하면 되는거 아닌가요?

라고 생각할 수도 있겠지만, 제 개인적인 생각으로는 네이버는 비밀번호를 쉽게 털리지 않을만한 보안 시스템이 어느정도 갖춰져 있으니까 영세할수록 길이라도 넉넉하게 해서 털릴 가능성을 줄이는게 맞다고 생각됩니다.

물론 사용자가 짧게 설정하면 의미가 없긴합니다. 네이버가 16글자로 제한하는 것에 대해 문제제기하고 공론화된적이 없는 것을 보면 대부분의 사람들은 16글자 이하로 설정하는게 아닐까 생각됩니다.

그럼 또 사람들 어차피 짧게 설정하는데 네이버처럼 최대 길이를 16글자로하면 되는거 아닌가? 라는 생각이 들기도 합니다. 그런데 비밀번호를 길게하고, 브라우저에 저장하고 다니는 1인으로써 저 같은 사용자를 위해 가능한 비밀번호 최대 길이를 늘리고 싶어집니다. 현실적으로는 보안적인 목적보다도 다양한 사용자의 요구를 만족시키기 위해 비밀번호 최대 길이를 최대한 늘리는 선택을 해야겠습니다. 현실적인 보안 목적을 달성하기 위해서라면 최소 길이를 늘리는게 합리적인 선택으로 보입니다.

참고로 해외에서는 100글자도 불만을 제기하는 글([링크](https://security.stackexchange.com/questions/169472/why-is-there-a-cap-on-password-length))이 올라오기도 합니다(답변은 질문자의 뇌피셜 비판하는 내용). 국내에서는 보안업무 하시는 것 같은 분들이 문제 제기하시는 글이 보이기도 합니다.

그러면 최대 길이 선택을 위한 자료를 마저 보겠습니다.

## OWASP Cheat Sheet

보안하면 빠지지 않고 등장하는 OWASP의 Cheat Sheet를 보겠습니다. [Authentication 페이지](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#implement-proper-password-strength-controls)에 조금 더 구체적인 지침이 등장합니다.

### Implement Proper Password Strength Controls

비밀번호 길이 부분만 살펴보면 아래와 같습니다. 마찬가지로 NIST 문서를 언급하면서 최소 8글자를 설정할 것을 언급합니다.

-   Password Length
    -   **Minimum**  length of the passwords should be  **enforced**  by the application. Passwords  **shorter than 8 characters**  are considered to be weak ([NIST SP800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html)).
    -   **Maximum**  password length should not be set  **too low**, as it will prevent users from creating passphrases. A common maximum length is 64 characters due to limitations in certain hashing algorithms, as discussed in the  [Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#maximum-password-lengths). It is important to set a maximum password length to prevent  [long password Denial of Service attacks](https://www.acunetix.com/vulnerabilities/web/long-password-denial-of-service/).

최대 길이에 관해서 언급하는 첫 문장에서는 최대 길이가 너무 짧으면 사용자가 `passphrases`를 하지 못할 수도 있다고 합니다.

passphrase는 비밀번호를 쉽게 기억하기 위해 A cat that loves dogs. 와 같은 의미있는 문장을 Acat th@tlov3sd0gs. 와 같이 특수문자를 포함한 문자열로 변경하여 비밀번호로 사용하는 것입니다(출처: [Skills for all by Cisco - Introduction to cybersecurity](https://skillsforall.com/course/introduction-to-cybersecurity)).

> 위 passphrase 예제가 별로 좋지 않은 것 같아 추가합니다.
>
> 보안업체인 Avast에 있는 [예제](https://www.avg.com/en/signal/how-to-create-a-strong-password-that-you-wont-forget)를 보면, passphrase는 `길이`가 길고, 관련없는 단어들을 조합하여 `무작위성`이 있어야 합니다.
>
> Broadways&Swimmer&Argue7&Pursuant&Dramatize  
> Rickety Output Oxidant Deem Spotless
>
> 그리고 `기억하기 쉬워야 한다`는 점을 강조합니다. 말도 안되는 문장이지만 본인만큼은 기억하기 쉬운 문장이라고 할 수 있겠습니다.

그리고 이 다음에 특정 해싱 알고리즘의 한계로 인해 `64글자를 최대 길이로 설정하는게 일반적`이라고 합니다. 일반적이라고는 하는데 저는 64글자 제한을 본적이 없어서 Password Storage Cheat Sheet를 살펴보니 bcrypt 때문인 것으로 보입니다.

> -   **For legacy systems using  [bcrypt](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#bcrypt), use a work factor of 10 or more and with a password limit of `72 bytes`.**

bcrypt 최대 길이를 벗어나는 경우 뒷 부분은 무시해버리는 질문 글([링크](https://security.stackexchange.com/questions/39849/does-bcrypt-have-a-maximum-password-length))도 참고자료로 있습니다. 최대 길이 설정을 하지 않는 경우, 최대 길이까지만 비밀번호가 같으면 같은 비밀번호가 됩니다.

마지막으로는 서비스 거부(Denial of Service) 공격을 방지하기 위해서 최대 길이를 설정해야 함을 언급합니다.

지금까지의 자료들을 종합해보자면, 이상적으로는 비밀번호 최대 길이를 제한하지 않는 것이 좋지만 너무 긴 길이의 비밀번호는 서비스 거부 공격에 악용되거나, 해싱 알고리즘의 길이 한계가 있을 수 있으니 최대 길이는 꼭 설정해야합니다.

### 해싱 알고리즘의 길이 한계가 없을 경우에는 어떻게 해야할까요?

아직 지식이 짧아서 뇌피셜로 작성해보겠습니다.

앞서 OWASP에서 언급한 Long password denial of service 페이지를 보면, 다양한 `비밀번호 길이별 응답 시간`을 비교하면서 취약점을 발견했다고 합니다.

> This vulnerability was detected by sending passwords with various lengths and comparing the measured response times.

이를 바탕으로 생각해볼 때, 사용하게 될 서버 사양에서 `비밀번호 길이별 응답 시간`을 체크하고, 요구사항에 따라 `수용할 수 있는 길이를 실험적으로 발견하고 설정`할 수 있을 것입니다.

## Outro

저 같은 경우 개인 프로젝트에 사용할 Spring Security의 기본 설정이 bcrypt 이고, 해싱 알고리즘을 자세히 살펴볼 여유가 없어서 일반적으로 사용하는 64글자를 비밀번호 최대 길이로 설정할 계획입니다. 하지만 bcrypt를 사용하더라도 라운드 수 설정 및 서버 성능에 따라 해싱 속도가 달라지는 것은 마찬가지이므로, 실험을 통해서 최대 길이를 더 줄일 가능성은 열어두어야겠습니다.

## 안전한 비번 만드는 비법, 보안에 도움 안된다

2017년 [ZDNET에 올라왔던 기사](https://zdnet.co.kr/view/?no=20170809110258)입니다.

> 자신만 알 수 있는 안전한 패스워드(password)를 만들어라. 알파벳 대소문자, 숫자, 특수문자를 섞어라. 주기적으로 바꿔라.

이 지침을 만들어서 NIST 문서에 반영했던 보안 전문가가 후회하고 개정된 문서에서는 이 내용이 제외됐다고 합니다.

> 많은 사람들이 이 규칙대로 복잡한 패스워드를 만들더라도 빈번한 변경 주기를 따르느라 문자열을 크게 바꾸지 않는데다 입력 자체를 불편하게 할 뿐이었다고 지적했다.

그리고 개정한 사람의 코멘트를 보면, 만료기간도 별 도움이 안된다고 합니다.

> 개정 작업을 맡은 NIST 표준 및 기술 고문 폴 그라시(Paul Grassi)는 특수문자 입력이나 만료기간같은 조건이 보안에 별로 도움이 안 됐다며 "실제로는 사용성에 부정적인 영향을 줬다"고 말했다.

[NIST 문서](https://pages.nist.gov/800-63-3/sp800-63b.html#-5112-memorized-secret-verifiers)를 조금 더 살펴보니 실제로 문자 조합 강제하지 말고, 주기적으로 변경을 요구할 필요도 없다고 합니다.

> Verifiers `SHOULD NOT impose` other composition rules (e.g., requiring mixtures of `different character types` or prohibiting consecutively repeated characters) for memorized secrets. Verifiers `SHOULD NOT require` memorized secrets `to be changed arbitrarily (e.g., periodically)`. However, verifiers SHALL force a change if there is evidence of compromise of the authenticator.

그리고 이 상황의 문제를 보여주는 만화

![xkcd](https://imgs.xkcd.com/comics/password_strength.png)  
출처: [xkcd](https://xkcd.com/936)

사용자가 원하는대로 문장으로 입력하는게 좋은 선택으로 보입니다. 문득 국내 비밀번호 지침은 언제 업데이트되는건가 궁금해집니다.