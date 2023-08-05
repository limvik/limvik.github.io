---
layout: post
title: Spring Boot로 REST API 만들어보기(2)-1 JWT
categories:
- etc
tags:
- JWT
date: 2023-08-05 15:43 +0900
---
## Intro

이전에 원티드 프리 온보딩 과제로 회원가입 기능을 만들면서 글([링크](/posts/making-rest-api-with-spring-boot-1-signup/))을 작성했었습니다. 이어서 로그인 기능을 만들면서 과정을 작성해보려 했는데... JWT 찾아보다가 글이 너무 길어져서 JWT 글이 됐습니다.

## 요구사항

먼저 이번에 구현할 로그인 요구사항을 살펴보겠습니다.

### 과제 2. 사용자 로그인 엔드포인트

- 사용자가 올바른 이메일과 비밀번호를 제공하면, 사용자 인증을 거친 후에 JWT(JSON Web Token)를 생성하여 사용자에게 반환하도록 해주세요.
- 과제 1과 마찬가지로 회원가입 엔드포인트에 이메일과 비밀번호의 유효성 검사기능을 구현해주세요.

유효성 검사 요구사항은 회원가입에서 사용한 로직을 그대로 따라가면 되니, 큰 어려움이 없겠습니다.

첫 번째 요구사항이 문제입니다. 사용자 인증에 성공하면 JWT 를 반환해야합니다. 문제는 JWT에 대해 아는게 거의 없다는 겁니다. JWT 부터 살펴봐야겠습니다.

## JWT(JSON Web Token)

### Token

이전에 Spring Security 를 공부하면서 Authentication 인터페이스의 구현으로 여러가지 토큰(Token)이 있던 것을 살펴본적([링크](/posts/study-spring-security-7-securitycontext-authentication-grantedauthority/#token))이 있습니다. 당시 예로 들었던 은행 서비스 이용 시 인증번호를 생성하는 OTP(One Time Password) 토큰을 생각해보면, 토큰이라는게 서버에서 내가 '나'라는 것을 식별할 수 있는 정보가 있다는 것을 알 수 있습니다. 그래서 정보 유출되지 않게 조심하라는 경고문구가 써있기도 하죠.

![OTP 토큰](/assets/img/2023-07-18-study-spring-security-7-SecurityContext-Authentication-GrantedAuthority/04-otp.jpg)

OTP 토큰은 OTP 토큰만의 방식이 있고, JWT는 JWT만의 방식이 있을테니 구체적으로 어떻게 되어있나 살펴보고 사용해야겠습니다.

### JWT는 어떻게 생겼을까?

글로만 먼저 보는것 보다 어떻게 생겼는지 먼저 보는게 좋을 것 같습니다. jwt 를 검색하면 항상 상위에 나타나는 [jwt.io](https://jwt.io/) 에서 JWT가 어떻게 생겼는지 먼저 보겠습니다.

![JWT 구조](/assets/img/2023-08-05-making-rest-api-with-spring-boot-2-1-jwt/01-jwt.png)

왼쪽의 `Encoded` 를 보면, `Decoded` 에 있는 HEADER를 알 수 없는 문자로 표현해서, `.` 을 찍은 후 PAYLOAD를 또 알수 없는 문자로 표현하고, 또 `.`을 찍은 후 VERIFY SIGNATURE를 마찬가지로 알 수 없는 문자로 표현합니다.

Base64가 보이는 것을 봐서는 Base64로 인코딩한 것으로 보입니다. 생긴거만 봐도 대충은 느낌이 오는 것 같습니다.

그러면 RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc7519))를 보면서 구체적으로 살펴보겠습니다.

### JWT의 발음

다른 내용보다 눈에 먼저 들어오더라구요...

>The suggested pronunciation of JWT is the same as the English word "`jot`".

읽기 편하긴 한데, 한국어가 모국어인 사람들에게는 공적인 자리에서 사용하기 좀 껄끄러운 발음입니다.

### Abstract 살펴보기

다음은 RFC 문서의 시작인 Abstract 부터 보겠습니다. 모르는걸 살펴보다 보면, 결국에는 다 살펴볼 수 밖에 없을 것 같습니다.

>JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties.
>
>JWT는 두 당사자 간에 전송해야 할 claims를 나타내는 compact하고 URL-safe한 수단입니다.

JWT의 첫 draft 문서([링크](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-json-web-token-00))를 보면 첫 문장이 아래와 같이 최종 문서와는 조금 차이가 있습니다.

>JSON Web Token (JWT) is a means of representing claims to be transferred between two parties.

첫 draft에 있는 내용은 `JWT는 두 당사자 간에 전송해야 할 claims를 표현하는 수단`이라고 하면서 조금 더 간단하게 설명하고 있습니다.

claims는 아무래도 앞에서 본 PAYLOAD에 있는 내용들일 것 같은 느낌이 오기는한데, compact 와 URL-safe 는 무엇을 의미하길래 추가된 건지 궁금해져서 먼저 살펴보겠습니다.

### Compact, URL-safe

먼저 Compact 라는 단어를 생각하면, 여자 화장품인 팩트가 compact라는 단어에서 비롯됐다는게 생각납니다. 간편하고, 휴대하기 쉬운 느낌이 있는 단어입니다.

RFC 문서를 조금 건너 뛰어서 Introduction([링크](https://datatracker.ietf.org/doc/html/rfc7519#section-1)) 섹션의 첫 문장에 아래와 같이 이야기하고 있습니다.

>JSON Web Token (JWT) is a `compact` claims representation format intended for space constrained environments such as HTTP Authorization headers and URI query parameters.
>
>HTTP Authorization headers 와 URI query parameters 같은 공간 제약이 있는 환경을 위한 compact한 claims 표현 형식입니다.

아직 경험이 부족해서 떠오르는게 없습니다... GPT를 소환할 때입니다.

claims 를 미리 간단하게 맛 보자면, payload에 있던 내용들이 claims 라고 합니다.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

이 claims 가 있는 JSON 을 그대로 Headers 에 포함시키기에는 형식이 맞지 않습니다. header는 `,` 로 값을 구분하므로, 위와 같은 경우 여러 개의 Authorization header가 있는 것이 됩니다.

예를 들어 서버에서 허용하는 HTTP method 를 알려주는 Allow([MDN 링크](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow))와 관련된 Spring 코드를 보면, `,`로 묶어서 보냅니다. 

Spring [HttpHeaders.java](https://github.com/spring-projects/spring-framework/blob/main/spring-web/src/main/java/org/springframework/http/HttpHeaders.java)

```java
public void setAcceptCharset(List<Charset> acceptableCharsets) {
    StringJoiner joiner = new StringJoiner(", ");
    for (Charset charset : acceptableCharsets) {
        joiner.add(charset.name().toLowerCase(Locale.ENGLISH));
    }
    set(ACCEPT_CHARSET, joiner.toString());
}
```

Spring Security 의 [CacheControlHeadersWriter.java](https://github.com/spring-projects/spring-security/blob/main/web/src/main/java/org/springframework/security/web/header/writers/CacheControlHeadersWriter.java) 는 하드 코딩을 통해 `,` 로 구분하고 있습니다.

```java
private static List<Header> createHeaders() {
    List<Header> headers = new ArrayList<>(3);
    headers.add(new Header(CACHE_CONTROL, "no-cache, no-store, max-age=0, must-revalidate"));
    headers.add(new Header(PRAGMA, "no-cache"));
    headers.add(new Header(EXPIRES, "0"));
    return headers;
}
```

URI도 예약된 문자가 있는 등([URI RFC 문서 링크](https://datatracker.ietf.org/doc/html/rfc3986#section-2.2)) 사용할 수 있는 문자를 제한하고 있으니 JSON 형식 그대로 사용할 수는 없습니다.

그래서 JWT는 앞서 봤듯 각각을 인코딩한 후 `.` 으로 구분해서 `HEADER.PAYLOAD.VERIFY SIGNATURE` 형태로 나타내는 형식을 사용합니다.

인코딩을 할 때 정확히는 `Base64url` 로 인코딩합니다. 다시 RFC 문서 6페이지로 가보면, 아래와 같이 언급하고 있습니다.

>A JWT is represented as a sequence of URL-safe parts separated by period ('.') characters.  Each part contains a base64url-encoded value.
>
>JWT는 마침표('.') 문자로 구분된 URL-safe parts의 시퀀스로 표시됩니다.  각 parts에는 base64url로 인코딩된 값이 포함됩니다.

둘의 차이점을 간단하게 설명하자면, Base64에는 URL에서 사용할 수 없는 문자가 포함돼서 url에서 사용할 수 있는 문자로만 구성해서 인코딩을 해줍니다.

> Base64url 인코딩 에서는 "+"는 "-"로, "/"는 "_"로 대체되며, 패딩 문자 "="는 제거됩니다.

`Base64 vs Base64url` 로 검색해보면 자료가 많이 나와서 더 궁금하신 분들은 찾아보시면 좋을 것 같습니다.

그리고 실제로는 아래와 같이 Header나 URL에서 사용할 수 있습니다.

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

GPT가 준 URL에서 사용한 예제는 아래와 같습니다. email 확인 링크를 보낼때 claims에 이메일 주소를 넣어 url query parameter를 통해 보내는 예제입니다.

![Example JWT in URL QUERY PARAMETER](/assets/img/2023-08-05-making-rest-api-with-spring-boot-2-1-jwt/04-jwt-in-url-query-parameter.png)

GPT가 링크만 줬는데, Decode 해보면 아래와 같은 JSON이 됩니다.

```json
{
  "email": "johndoe@example.com",
  "iat": 1605175892,
  "exp": 1605183092
}
```

팀 프로젝트 할 때 이메일 인증 코드로 UUID를 사용하고, 데이터베이스에서 EVENT 등록을 했었습니다. JWT를 썼으면 데이터베이스에 굳이 EVENT로 부하도 안주고 좋았을 것 같습니다.

이렇게 URL에서 사용가능한 문자로만 만들어 문제없이 사용할 수 있는 것을 URL-safe 하다고하며, Compact는 HTTP headers 나 URI 쿼리 파라미터 같이 제약이 있는 곳에서도 사용할 수 있게 만들어주는 것이라고 이해할 수 있었습니다.

Compact 하게 만들다보면, 자연스럽게 URL-safe 가 따라올 수 밖에 없어 보입니다.

URL과 URI를 혼용해서 용어를 막 사용하기는 했지만, URL은 URI의 하위 개념이므로 받아들이는데 큰 문제가 없을 것으로 생각됩니다.

다음은 claims 를 살펴보겠습니다.

### Claims

앞에서 claims 는 이런거다, 하면서 봤었습니다. 

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

두 당사자 간에 전달하고자 하는 정보라고 막연하게 생각해볼 수 있겠습니다.

RFC 문서의 용어 설명(Terminology)을 살펴보겠습니다.

> `Claim`: A piece of information asserted about a subject.  A claim is represented as a name/value pair consisting of a Claim Name and a Claim Value.
>
>주체에 대해 주장된?, 진술된? 정보의 조각입니다. claim은 Claim Name 과 Claim Value로 구성된 이름/값 쌍의 형태로 표현됩니다.

asserted 를 한글로 표현하기가 매끄럽게 표현하기가 어렵습니다. JSON 형식에서 "sub": "1234567890" 과 같이 표현된 데이터를 claim 이라고 한다는 것을 알 수 있습니다. Claim Name 과 Claim Value 는 당연하게도 파악이 되지만, 그래도 용어 설명이 같이 붙어있으니 간단하게 보고 넘어가겠습니다.

>`Claim Name`: The name portion of a claim representation.  A Claim Name is always a `string`.
>
>claim 표현의 name 부분으로, Claim Name은 항상 문자열(string) 입니다.

JSON 형식이고, JavaScript의 Object 는 key 로 문자열만 사용해야하니 당연한 이야기를 하고 있습니다. JavaScript 를 전혀 사용해보지 않으신 분들은 참고해볼만 하겠습니다([JSON Literals](https://www.w3schools.com/js/js_json_objects.asp)).

>`Claim Value`:  The value portion of a claim representation.  A Claim Value can be any JSON value.
>claim 표현의 value 부분으로, Claim Value 는 JSON value의 어떤 value이든 될 수 있습니다.

진짜 뭐든지 될 수 있나 궁금해서 JSON RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc8259#section-3))를 보니, `undefined`는 목록에 없습니다.

```
A JSON value MUST be an object, array, number, or string, or one of the following three literal names:

      false
      null
      true

The literal names MUST be lowercase.  No other literal names are allowed.

      value = false / null / true / object / array / number / string

      false = %x66.61.6c.73.65   ; false

      null  = %x6e.75.6c.6c      ; null

      true  = %x74.72.75.65      ; true
```

false, null, true는 반드시 소문자로 사용해야 한다는 점 주의해야겠습니다. object, array, number, string 에 대해서 자세히 설명되어 있어 필요할 때 참고하는 것도 좋겠습니다.

구현적인 측면에서 보자면, JavaScript Object 에서 하나의 Property 를 Claim 이라고 하고, key 를 Claim Name, value 를 Claim Value 라고 합니다.

RFC 문서의 용어 설명에 연관된 것들을 몇 가지 더 보겠습니다.

>`JWT Claims Set`: A JSON object that contains the claims conveyed by the JWT.
>
>JWT가 전달하는 claims가 포함된 JSON 객체입니다.

앞에서 봤던 claims 가 포함된 JSON 객체의 정확한 명칭은 `JWT Claims Set` 이라고 한다는 것을 알 수 있습니다.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

RFC 문서에서 JWT Claims Set 바로 위에 JWT 용어 설명이 있어 보고 넘어가겠습니다.

>`JSON Web Token (JWT)`: A string representing a set of claims as a JSON object that is encoded in a JWS or JWE, enabling the claims to be digitally signed or MACed and/or encrypted.

또 새로운게 등장합니다. JWT Claims Set이 JWS 또는 JWE 로 인코딩된 문자열 표현이라고 합니다. claims 에는 전자 서명 또는 MAC를 적용 하거나 혹은 암호화까지 함께 수행한다고 합니다.

JWT 는 단순히 Base64url로 인코딩한 것으로 끝나는게 아니라 무결성 또는 기밀성을 보장하기 위한 조치까지 수행해야 한다는 것을 알 수 있습니다. [jwt.io](https://jwt.io/) 에서 VERIFIED SIGNATURE 에 알고리즘을 적용하고 있다는 것을 보기도 했었는데, 이와 관련된 내용으로 보입니다.

지금은 claims 를 보고 있으므로, claims 를 마저 보고 또 살펴보겠습니다.

### Claims의 종류

RFC 문서를 보면 Claims의 종류로 세 가지를 제시하고 있습니다.

- Registered Claim Names
- Public Claim Names
- Private Claim Names

정확히는 Claim Name의 종류라고 할 수 있겠습니다.

문서에서 제시하고 있는 Registered Claim Names의 종류는 아래와 같습니다.

- "iss" (Issuer) Claim
- "sub" (Subject) Claim
- "aud" (Audience) Claim
- "exp" (Expiration Time) Claim
- "nbf" (Not Before) Claim
- "iat" (Issued At) Claim
- "jti" (JWT ID) Claim

제일 처음에 봤던 [jwt.io](https://jwt.io/)의 PAYLOAD 예제에 있던 Claim Name이 보입니다.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

`name`은 안보이는데 이는 Public Claim Names 목록([링크](https://www.iana.org/assignments/jwt/jwt.xhtml))에 있으며, IANA에서 관리합니다.

Private Claim Names 는 이름에도 알 수 있듯 공식적인 Claim은 아니지만, 당사자간에 합의하에 Claim Names 를 지정해서 사용할 수 있도록하고 있습니다.

앞에서 예제로 사용했던 것 중에 sub와 admin은 목록에서 찾아볼 수 없는데, 이런 Claim Name을 Private Claim Names라고 칭할 수 있겠습니다.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

RFC 문서에서는 Private Claim Names 사용 시 Registered 혹은 Public Claim Names 와 충돌하지 않도록 주의하라고 합니다.

그런데 Registered Claim Names와 Public Claim Names 목록을 살펴봐도 [jwt.io](https://jwt.io/) 에서 봤던 HEADERS 의 JSON 에 있는 것은 안보입니다.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

JWT RFC 문서의 table of contents 를 보면 Claims 바로 뒤에 JOSE headers 섹션의 하위 섹션으로 "typ"이 있는 것이 보입니다. HEADERS 는 Claims 에 속하는게 아니라는 것을 알 수 있습니다.

정리해보자면 PAYLOAD 에 속한 JSON Object 의 set of claims 가 `JWT Claims Set` 이라고 할 수 있겠습니다. Claim 이라는 단어의 뜻([네이버 사전 링크](https://en.dict.naver.com/#/entry/enko/876b1c693dc145a887d2017c5e9fe7d6))을 생각해볼 때, header 에 있는 알고리즘이나 타입 등의 정보가 특정 행위(정보 접근 등)를 위한 주된 정보는 아니므로 구분되어지는 것 같습니다.

그럼 또 JOSE headers 는 무엇인가 살펴봐야겠습니다.

### JOSE headers

RFC 문서에 JOSE 가 무엇의 약자인지 나와있지를 않아서, IETF 웹 사이트([링크](https://datatracker.ietf.org/wg/jose/about/))에 있는 것을 찾아봤습니다.

> Javascript Object Signing and Encryption

간단하게 설명을 살펴보면 JWT에 사용하려고 정의했던 건데 여기저기 널리 쓰이고 있고, 원래 JOSE working group 은 JWS, JWE, JWK, JWA 등 JSON 기반 표현을 표준화하는 working group 이었다고 합니다. 나머지 내용은 현재의 JOSE와 JWT에 정의된 컨테이너로는 영지식증명(ZKP, Zero-Knowledge Proofs) 알고리즘을 사용하여 데이터를 표현할 수 없어 이와 관련된 작업을 한다고 합니다. 마일스톤에 7월 중 JSON Web Proof 초안을 채택한다고 되있는데, 미뤄지나 봅니다.

영지식증명은 보안기사 공부할 때 스치듯 한 문제를 봤던 것 같은데, 여기로 빠지면 또 먼길 돌아가야하니 묻어두도록 하겠습니다.

다시 돌아와서 JWT 문서에 있는 JOSE headers 섹션([링크](https://datatracker.ietf.org/doc/html/rfc7519#section-5))설명을 보겠습니다.

>For a JWT object, the members of the JSON object represented by the JOSE Header describe the cryptographic operations applied to the JWT and optionally, additional properties of the JWT.  Depending upon whether the JWT is a JWS or JWE, the corresponding rules for the JOSE Header values apply.
>
> JWT에 적용하는 암호화 작업과 JWT의 선택적, 추가적 properties 를 설명하는 JOSE header에 의해 JSON object로 표현된 멤버들을 의미합니다. JWT가 JWS 또는 JWE 인지 여부에 따라 JOSE Header value 에 해당하는 규칙이 적용됩니다.

첫 번째 문장은 앞에서 봤던 HEADER를 글로 설명하고 있습니다. 그리고 HEADER에 적용되는 value를 통해 JWS 혹은 JWE 인지 여부를 판단할 수 있겠습니다. 그런데 저는 잘 모르겠습니다.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

나머지 내용도 마저 보겠습니다.

>This specification further specifies the use of the following Header Parameters in both the cases where the JWT is a JWS and where it is a JWE.
>
>이 사양은 JWT가 JWS인 경우와 JWE인 경우 모두에서 다음 Header Parameters의 사용을 추가로 지정합니다.

RFC 문서에서 JOSE header 하위 섹션에는 "typ" 은 Media Type([IANA Media Type 목록](https://www.iana.org/assignments/media-types/media-types.xhtml)) 이고, 위 예제에는 없지만 "cty"를 포함해서 "typ" 은 JWS 와 JWE 에 의해 정의된다고 합니다. JWE 로 사용 시에는 PAYLOAD 에 있는 claim을 HEADER로 복제해서 사용하는 방식도 사용한다는 설명이 있기도 합니다. 그런데 "alg"는 안보입니다.

algorithm의 약자라는건 유추 가능하지만, 서명 알고리즘과 암호화 알고리즘의 차이에 대한 지식이 없어서 그런지 뭐 어떻게 "typ" 이 JWS, JWE 여부에 따라 결정된다는 건지 잘 모르겠습니다.

그래서 JWS의 RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.1))와, JWE의 RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc7516#section-4.1.1))를 보니 둘다 "alg" header parameter 에 대한 설명이 나와 있고, 이외에도 여러 header parameter 를 정의하고 있습니다. 그리고 다시 "alg" header parameter의 설명을 가보면 둘 다 `JWA(JSON Web Algorithm)`의 RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc7518))로 가보라고 합니다.

JWA RFC 문서에는 JWS 용 "alg" header parameter 섹션([링크](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1)) 과 JWE용 섹션([링크](https://datatracker.ietf.org/doc/html/rfc7518#section-4.1))이 나뉘어져 있고, 적용되는 알고리즘의 header value가 달라 "alg"의 value가 JWS, JWE 여부에 따라 달라진다는 것을 알 수 있습니다. 너무 길어서 알고리즘 목록을 여기에 추가하지는 않겠습니다. 추천하는 알고리즘을 표시해두고 있어서 알고리즘 선택 시에 참고하면 좋을 것 같습니다.

다시 예제를 보면 "alg" header parameter에 적용된 value를 보고, JWS 임을 알 수 있습니다.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

다시 JWT RFC 문서로 돌아와서 Overview 섹션(링크)에 있는 JOSE Header 관련 내용을 살펴보면서 마무리하겠습니다.

>The contents of the JOSE Header describe the cryptographic operations applied to the JWT Claims Set.  If the JOSE Header is for a JWS, the JWT is represented as a JWS and the claims are digitally signed or MACed, with the JWT Claims Set being the JWS Payload.  If the JOSE Header is for a JWE, the JWT is represented as a JWE and the claims are encrypted, with the JWT Claims Set being the plaintext encrypted by the JWE. A JWT may be enclosed in another JWE or JWS structure to create a Nested JWT, enabling nested signing and encryption to be performed.
>
>JOSE Header의 내용은 JWT Claims Set에 적용되는 암호화 작업을 설명합니다.  JOSE Header가 JWS용인 경우 JWT는 JWS로 표현되며 claims는 디지털 서명 또는 MAC 처리되며, JWT Claims Set은 JWS Payload가 됩니다.  JOSE Header가 JWE용인 경우 JWT는 JWE로 표시되고 claims는 암호화되며, JWT Claims Set은 JWE에 의해 암호화된 평문이 됩니다. JWT는 중첩된 서명 및 암호화를 수행할 수 있도록 다른 JWE 또는 JWS 구조로 둘러싸여 Nested JWT를 생성할 수 있습니다.

제일 처음 봤던 JWT 예제는 JWS 예제라고 할 수 있겠습니다. 만약 정말 중요한 비밀정보가 포함된다면 JWE 로 암호화를 하는게 좋겠습니다. 일부만 암호화가 필요할 경우에는 JWE를 JWS에 중첩해서 포함하는 방법도 고려해볼만 하겠습니다.

JWT가 어떻게 구성되어 있는지와 JWT를 이해하기 위한 용어는 어느정도 살펴본 것 같아 RFC 문서의 Introduction 을 살펴보면서 정리하겠습니다.

### RFC 문서 Introduction 살펴보기

앞에서 본 문장과 내용들이라 따로 번역을 붙이지는 않겠습니다. 처음에 볼 때는 compact claims 부터 막혔었는데, 이제는 그래도 어느정도 큰 거부감 없이 읽혀집니다.

>JSON Web Token (JWT) is a compact claims representation format intended for space constrained environments such as HTTP Authorization headers and URI query parameters.  JWTs encode claims to be transmitted as a JSON [[RFC7159](https://datatracker.ietf.org/doc/html/rfc7159)] object that is used as the payload of a JSON Web Signature (JWS) [[JWS](https://datatracker.ietf.org/doc/html/rfc7519#ref-JWS)] structure or as the plaintext of a JSON Web Encryption (JWE) [[JWE](https://datatracker.ietf.org/doc/html/rfc7519#ref-JWE)] structure, enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted.  JWTs are always represented using the JWS Compact Serialization or the JWE Compact Serialization.

또 파고들자면 전자서명, MAC 는 무엇인지 부터 시작해서 파고들 수 있겠지만, 별도로 다뤄봐야 할 주제인 것 같고, 지금은 구현 스케줄도 빡빡하니 다음으로 미루겠습니다.

마지막 문장에 나와있듯 JWT는 항상 JWS 아니면 JWE를 사용해서 표현된다는 점에 주의해야겠습니다.

### 다시 보는 JWS

지금까지 용어들을 살펴보면서 JWT 구조 정도는 이해할 수 있게 됐습니다. JWS 인줄도 모르고 봤던 [jwt.io](https://jwt.io/)에 있는 예제를 다시 보겠습니다.

![Encoded and Decoded JWS](/assets/img/2023-08-05-making-rest-api-with-spring-boot-2-1-jwt/02-jwt-terminology.png)

이제 왜 인코딩을 했는지(Compact, URL-safe), HEADER에는 무엇이 들어가는지(JOSE Headers), PAYLOAD에는 무엇이 들어가는지(JWT Claims Set) 혹은 어디서 찾아볼 수 있는지 등을 조금 알 수 있게 됐습니다. 그리고 VERIFY SIGNATURE는 HEADER나 PAYLOAD 에 있는 내용이 변경됐는지 확인할 수 있도록(무결성) 전자서명 또는 MAC를 적용한 결과라는 점을 어렴풋이나마 알 수 있었습니다.

### JWE 예제

JWE 는 JWS 구조와 다릅니다. 그림은 4를 건너 뛰었는데, 출처를 가보시면 알 수 없는 무언가(JWE AAD)를 추가해두기는 했습니다. Header도 Base64url 로 인코딩을 해야하는데, 빠져있기도 합니다.

![JWE](/assets/img/2023-08-05-making-rest-api-with-spring-boot-2-1-jwt/03-jwe-example.webp)

출처: [https://0xn3va.gitbook.io/cheat-sheets/web-application/json-web-token-vulnerabilities](https://0xn3va.gitbook.io/cheat-sheets/web-application/json-web-token-vulnerabilities)

지금 쓰지는 않을 것 같아서, JWT 를 검색하면 블로그에서 자주 보이는 것은 JWS 형식이고, JWE 형식은 다르다는 걸 인지하고 마무리해야겠습니다.

## Outro

순서대로 안보고, 궁금한 순서대로 봤더니 좀 뒤죽박죽인 것 같습니다. 그래도 JWT 구조를 파악하고, 용어를 이해할 수 있는 시간이었습니다.

이제 구현을 해야 하는데... 단순히 JWT 구조 좀 알았다고 구현을 할 수는 없죠. 팀 프로젝트도 해야하다보니 투자할 수 있는 시간이 많이 안나서 아쉽습니다. 구현이야 라이브러리 사용하고 예제를 복붙하든 어쩌든 할 수 있겠지만, 그럼 저에게 별 의미없는 시간이 될 테니, 빠른 제출 가산점은 포기하고, 궁금한거 공부하면서 기한 내에 제출하도록 계획을 변경해야겠습니다.

## 참고 자료
- RFC 7519 - JWT([링크](https://datatracker.ietf.org/doc/html/rfc7519))
- RFC 8259 - JSON([링크](https://datatracker.ietf.org/doc/html/rfc8259))
- JWT Public Claim Names([링크](https://www.iana.org/assignments/jwt/jwt.xhtml))
- JSON Web Tokens as Building Blocks for Cloud Security([링크](https://www.ibm.com/blog/json-web-tokens-as-building-blocks-for-cloud-security/))
- Token Authentication([링크](https://www.logintc.com/types-of-authentication/token-authentication/))
- jwt.io([링크](https://jwt.io/))
- [JWT] 토큰 기반 인증에 대한 소개([링크](%5BJWT%5D%20%ED%86%A0%ED%81%B0%28Token%29%20%EA%B8%B0%EB%B0%98%20%EC%9D%B8%EC%A6%9D%EC%97%90%20%EB%8C%80%ED%95%9C%20%EC%86%8C%EA%B0%9C))
- Spring Security JWT 토큰으로 인증하기([링크](https://petaverse.pe.kr/entry/Spring-Security-JWT-%ED%86%A0%ED%81%B0%EC%9C%BC%EB%A1%9C-%EC%9D%B8%EC%A6%9D%ED%95%98%EA%B8%B0?category=1113161))
- String based data encoding: Base64 vs Base64url([링크](https://stackoverflow.com/questions/55389211/string-based-data-encoding-base64-vs-base64url))
- 구글 맵 URL 인코딩([링크](https://developers.google.com/maps/url-encoding?hl=ko))
