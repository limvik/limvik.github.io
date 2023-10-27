---
layout: post
title: 쿠키(Cookie) 설정 속성 살펴보기
categories:
- Web
tags:
- Cookie
- Security
- XSS
- CSRF
- Session Fixation
date: 2023-10-22 18:57 +0900
---
## Intro

쿠키(Cookie)는 보안 상 좋지 않다는 이야기를 자주 듣다보니, 쿠키를 어떻게든 사용하지 않고 외면(?)해 보려고 했습니다.

그런데 쿠키의 단점을 해소하기 위해 만들어졌다는 세션 방식도 세션 쿠키를 만들어서 세션 아이디를 저장합니다. 그리고 세션의 단점을 해소하기 위해 만들어졌다는 토큰 방식도 패턴에 따라 다르기는 하지만, 어쨌든 쿠키에 관련 정보를 저장합니다.

쿠키의 굴레에서 탈출할 수 없다면, 잘 쓰는 수 밖에 없겠죠? 그동안 혼자서 외면해왔던 쿠키의 기본적인 정보부터 살펴보려고 합니다.

특별한 내용 없이 MDN [Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) 문서와 [Using HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) 문서를 기반으로 관련 정보와 함께 살펴봅니다.

## Overview

쿠키는 웹(Web) 쿠키, 인터넷(Internet) 쿠키, 브라우저(Browser) 쿠키 등 다양하게 불리고, 구체적으로는 목적이나 속성에 따라 Authentication 쿠키, Session 쿠키, Persistent 쿠키, Tracking 쿠키 등 으로 부르기도 합니다.

### 어원

위키피디아의 [HTTP Cookie](https://en.wikipedia.org/wiki/HTTP_cookie) 문서에 따르면, Netscape Communications에 근무하던 [Lou Montulli](https://en.wikipedia.org/wiki/Lou_Montulli)가 UNIX 프로그래머들이 프로그램 간에 데이터를 주고받을 때 사용하던 작은 데이터 패킷인 [Magic Cookie](https://en.wikipedia.org/wiki/Magic_cookie)를 사용하자는 아이디어를 내면서 Cookie라는 용어가 유래되었다고 합니다. 참고로 UNIX에서는 로그인/로그아웃 시에 표시되는 명언이나 농담을 저장하는 [Fortune Cookie](http://catb.org/jargon/html/F/fortune-cookie.html)도 있다고 합니다.

어떤 사람이 Lou 에게 직접 Cookie라는 용어를 어디서 가져온거냐고 물어봤더니, Magic Cookie 에서 가져왔다고 했다고 합니다([링크](https://dominopower.com/article/where-cookie-comes-from/)). 당시 Lou가 같이 보낸 사전(dictionary) 링크에 정의된 Magic Cookie 설명은 아래와 같습니다.

> Something passed between routines or programs that enables the receiver to perform some operation.
>
> 수신자가 어떤 작업을 수행할 수 있도록 루틴이나 프로그램 사이에 전달되는 것

사전의 정의가 생각보다는 잘 와닿지 않는 것 같아서 Fortune Cookie가 정의되어 있던 사이트에서 든 예시([링크](http://catb.org/jargon/html/C/cookie.html))를 보니, 더 잘 와닿는 것 같습니다.

> 세탁소에서 받은 claim check는 쿠키의 완벽한 일상적인 예이며, 이 쿠키가 유용한 유일한 용도는 나중에 이 거래와 관련된 거래(동일한 옷을 돌려받을 수 있도록)를 하는 것입니다.

### 탄생 배경

Netscape Communications에 근무하던 Lou가 쿠키를 제안한 것은 Lou가 고객사의 이커머스 어플리케이션을 개발하고 있었기 때문입니다. Lou는 고객사로부터 Server에 거래 정보 중 일부를 저장하고 싶지 않다는 의견을 듣고, `장바구니 정보를 저장`하기 위한 솔루션으로 제공되었다고 합니다. 실제로 처음 쿠기가 사용된 것은 `사용자 방문 여부를 저장`하기 위해 사용됐다고 합니다.

그리고 이러한 쿠키 탄생의 기술적인 배경에는 HTTP가 `stateless` 하다는 점이 있습니다.

### 크기 및 갯수 제한

RFC 문서([RFC 6265](https://datatracker.ietf.org/doc/html/rfc6265#section-4.1.1))에 따르면 쿠키의 크기 및 갯수 제한은 아래와 같습니다.

- At least `4096 bytes` per cookie (as measured by the sum of the length of the `cookie's name, value, and attributes`).
- At least `50` cookies per `domain`.
- At least `3000 cookies total`.

> 이틀 전(2023-10-20) 발행된 문서([RFC 6265bis](https://httpwg.org/http-extensions/draft-ietf-httpbis-rfc6265bis.html#name-limits))를 글 쓰고나서 발견해서 보니, 크기에 대한 제한은 없어졌습니다.

`At least` 이므로 `최소한`의 기준만 제시하고 있습니다.

구체적인 구현 방식은 브라우저마다 달라질 수 있기 때문에, 필요하다면 브라우저 마다 어떻게 제한을 하고 있는지 확인할 필요가 있겠습니다.

예를들어 Chrome의 경우 [chromestatus.com](https://chromestatus.com/feature/4946713618939904)에 올라온 글을 살펴보면, 속성 값(Attirbute Value) 크기에 대한 제한을 하기도 합니다.

> After the spec change corresponding to this Intent, user agents are now required to limit the sum of the lengths of the cookie's name and value to 4096 bytes, and `limit the length of each cookie attribute value to 1024 bytes`. Any attempt to set a cookie exceeding the name+value limit is rejected, and any cookie attribute exceeding the attribute length limit is ignored.

그리고 진실여부를 검증해보지는 않았지만, 브라우저별로 다른 제한이 있음을 구글 검색 결과를 통해 알 수 있습니다.

![구글에서 검색 결과](/assets/img/2023-10-22-cookie-attributes/01.cookie-limit.png)

MDN 문서에서 제안하는대로 Web Storage API(localStorage, sessionStorage) 또는, 사용하는 사이트를 본적은 없지만 IndexedDB 도 있으니 제한으로 인한 문제가 생길 일은 거의 없지 않을까 생각됩니다.

## 쿠키 생성 및 사용

서버가 HTTP 요청(request)을 클라이언트로부터 받았을 때 `Set-Cookie` 헤더를 설정하여 응답(response)하게 되면, 클라이언트 저장소에 쿠키가 저장됩니다. 그리고 일반적으로 같은 서버로 클라이언트에서 요청할 때 `Cookie` 헤더에 쿠키에 저장된 정보를 포함하여 보내게 됩니다.

### Set-Cookie Syntax

```text
Set-Cookie: <cookie-name>=<cookie-value>
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<number>
Set-Cookie: <cookie-name>=<cookie-value>; Partitioned
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
Set-Cookie: <cookie-name>=<cookie-value>; Secure

Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=None; Secure

// Multiple attributes are also possible, for example:
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly
```

Set-Cookie의 사용 예제를 살펴보면 아래와 같습니다.

### Set-Cookie

```text
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[page content]
```

그리고 클라이언트에서 Cookie 헤더에 Cookie 정보를 설정하여 서버로 보내는 예제는 아래와 같습니다.

### Cookie

```text
GET /sample_page.html HTTP/2.0
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

## 쿠키 설정

이 글을 읽고 있다면 쿠키는 보안에 취약하다는 이야기는 한 번쯤 들어보셨을 겁니다. 이러한 단점을 보완하기 위한 설정이 다양하게 존재합니다.

Chrome 개발자 도구에서 쿠키를 확인해보면, 접속한 사이트에서 사용하는 쿠키에 어떤 설정들이 되어있는지 확인할 수 있습니다.

아래는 Github 에서 개인 프로필 페이지에 접속했을 때의 쿠키 목록입니다.

![Github 프로필 페이지 접속 시 개발자 도구 쿠키 화면](/assets/img/2023-10-22-cookie-attributes/02.developer-tools-in-chrome.png)

이제 MDN 문서에 나온 각 설정과 관련된 내용을 살펴보겠습니다.

### Cookie Name and Value

당연한 이야기지만 쿠키의 Name과 Value는 필수적으로 설정해야합니다.

> Set-Cookie: yummy_cookie=choco

앞의 예제를 보자면 yummy_cookie 가 Name이고, choco는 Value가 됩니다.

#### Name

Name 에는 제어 문자(ASCII 문자 0 ~ 31과 127번 문자)와 구분 문자(스페이스, 탭 그리고 `( ) < > @ , ; : \ " / [ ] ? = { }`)를 제외한 ASCII 문자를 사용할 수 있습니다.

> 참고)  
> [Wikipedia](https://en.wikipedia.org/wiki/ASCII)에 따르면 ASCII 문자 32번은 SPACE 인데, printable 하므로 제어 문자로 취급하지 않습니다.

#### Value

Value는 선택적으로 큰 따옴표(double quote)를 사용하여 값을 감쌀 수 있습니다. 그리고 Name과 비슷한 제약이 있습니다.

ASCII 제어 문자와 [`Whitespace`](https://developer.mozilla.org/en-US/docs/Glossary/Whitespace) 그리고 `" , ; \`를 제외한 ASCII 문자를 사용할 수 있습니다.

Whitespace는 스페이스, 탭, Line Feed 등 공백을 의미하는 모든 것을 의미합니다.

제약이 있어 한글을 쓴다거나 하려면 몇 가지 과정을 거쳐야겠지만, localStorage가 있는데 굳이 쿠키에 한글을 저장하지는 않을 것 같습니다.

#### Name prefix

쿠키 Name에는 prefix를 지정하여 특정한 속성을 설정해야 함을 강제할 수 있습니다.

```text
// Both accepted when from a secure origin (HTTPS)
Set-Cookie: __Host-ID=123; Secure; Path=/
Set-Cookie: __Secure-ID=123; Secure; Domain=example.com

// Rejected due to missing Secure attribute
Set-Cookie: __Secure-id=1

// Rejected due to the missing Path=/ attribute
Set-Cookie: __Host-id=1; Secure

// Rejected due to setting a Domain
Set-Cookie: __Host-id=1; Secure; Path=/; Domain=example.com
```

##### __Host-  

쿠키 Name에 `__Host-` 가 있는 경우, `Secure` 속성이 표시되어 있고, secure origin에서 전송되었으며, `Domain` 속성이 포함되지 않고, `Path` 속성이 `/`로 설정된 경우에만 `Set-Cookie` 헤더에서 쿠키가 허용됩니다. 이렇게 하면 이러한 쿠키는 "도메인 잠금" 쿠키로 볼 수 있습니다.

##### __Secure-

쿠키 이름에 `__Secure-`가 있는 경우 Secure 속성이 표시되어 있고 secure origin에서 전송된 경우에만 `Set-Cookie` 헤더에서 쿠키가 허용됩니다. `__Secure-`는 `__Host-` 보다 `약`합니다.

---

정리해 보자면, Name prefix에는 `__Secure-` 와 `__Host-` 두 가지 prefix가 있는데, 모두 `Secure` 속성을 설정해야 합니다.

`__Secure-`는 `Domain` 속성을 반드시 설정해야하고, `__Host-` 는 `Path=/`을 설정(`/` 까지 필수)해야 합니다. `__Host-`에는 `Domain`을 설정하면 안됩니다.

그런데 왜 Prefix를 사용해야 하는걸까요?

MDN 문서에서 언급된 문제로는 `Session Fixation` 공격이 있습니다. Session Fixation은 공격자가 미리 설정한 세션 ID를 희생자에게 강제로 사용하게 만드는 공격입니다. 사용자가 공격자에 의해 전달된 세션 ID를 이용하여 로그인 하게되면, 공격자 또한 해당 세션 ID를 가지고 서버에 요청을 할 수 있게됩니다. Session Fixation은 살펴보니 간단하게 다룰만한 내용이 아니어서, 잘못된 지식이 전달될 확률이 높아 자세한 설명은 생략하겠습니다.

지금은 의문이 생기는게 너무 많아서 제 수준에서 이해할만한 공격과 방어 예시를 찾게되면, 추가 업데이트하겠습니다. 지금은 `Name Prefix를 이용해서 Session Fixation을 방어할 수 있다.` 정도로 넘어가겠습니다.

다음으로 Domain과 Path를 이어서 살펴보겠습니다.

### Domain

```text
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
```

`Domain`은 `쿠키를 수신할 수 있는 호스트`를 지정합니다. 현재 도메인만 값으로 설정하거나 공개 접미사([public suffix](https://publicsuffix.org/))가 아닌 경우 상위 도메인을 설정할 수 있습니다. 이전 사양(specifications)과 달리 도메인 이름(**.**example.com)의 선행 점(example 앞의 .)은 무시됩니다.

github의 경우 Domain에 github.com 인 것도 있고 .github.com인 것도 있는데, 변경된 사양을 반영하지 않은 것으로 보입니다.

![Github Domain 설정](/assets/img/2023-10-22-cookie-attributes/03.cookie-domain.png)

여러개의 호스트/도메인 값은 허용되지 않지만 `도메인을 지정`하면 `하위 도메인은 항상 포함`됩니다. 예를 들어 Domain=mozilla.org로 설정하면 developer.mozilla.org와 같은 하위 도메인에서 쿠키를 사용할 수 있습니다.
  
이 속성을 `생략`하면 기본적으로 이 속성은 `하위 도메인을 포함하지 않고` 현재 문서 URL의 호스트가 됩니다. 따라서 생략하는 것이 더 제한적입니다.

### Path

```text
Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
```

브라우저가 Cookie 헤더를 전송하기 위해 요청된 URL에 반드시 존재해야 하는 경로(path)를 나타냅니다.  

슬래시(/) 문자는 디렉토리 구분 기호로 해석되며 하위 디렉터리와도 매칭됩니다. 예를 들어 `Path=/docs`, 의 경우 요청 경로 `/docs`, `/docs/`, `/docs/Web/`, `/docs/Web/HTTP`가 모두 `일치`합니다.  반면에, 요청 경로 `/`, `/docsets`, `/fr/docs`는 일치하지 않습니다.

`/` 로 설정한다면 모든 경로에서 사용한다는 것을 의미하게 됩니다.

### Expires / Max-Age

```text
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<number>
```

`Max-Age`는 쿠키가 만료될 때까지의 `시간(초)`을 나타냅니다. `0` 또는 `음수`인 경우 쿠키가 `즉시 만료`됩니다. Expires와 Max-Age가 모두 설정되어 있으면 `Max-Age가 우선`합니다.

`Expires`는 쿠키의 최대 수명을 HTTP-[date](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date) 타임스탬프로 표시한다고 MDN 문서에 나와있습니다.

> Date: Wed, 21 Oct 2015 07:28:00 GMT

그런데 Github, Google 등 에서 쿠키를 확인해보면, ISO 8601 방식으로 표시됩니다.

> 2024-09-28T15:16:26.000Z

RFC 문서([링크-3.1 Examples](https://datatracker.ietf.org/doc/html/rfc6265#section-3.1)) 상에서는 MDN 문서에서 언급한 형식을 사용한 것으로 보아, 브라우저마다 구현이 다를 수 있겠습니다.

>   Set-Cookie: lang=en-US; Expires=Wed, 09 Jun 2021 10:18:14 GMT

`Expires`를 `지정하지 않으면 쿠키는 세션(Session) 쿠키`가 됩니다. `클라이언트가 종료`되면 세션이 종료되고 그 후 `세션 쿠키가 제거`됩니다. Expires가 설정된 경우 `기한은` 서버가 아닌 쿠키가 설정되는 `클라이언트를 기준`으로 합니다.
  
> 대부분의 웹 브라우저에는 모든 탭을 저장하고 다음에 브라우저를 사용할 때 복원하는 `세션 복원 기능`이 있습니다. `세션 쿠키도` 마치 브라우저를 닫지 않은 것처럼 `복원`됩니다.  

### HttpOnly

XSS(Cross Site Script) 공격을 방지하기 위해, JavaScript가 Document.cookie 속성 등을 통해 쿠키에 액세스하는 것을 금지합니다.

```javascript
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

앞서 보안상 별로 중요하지 않은 정보인 경우 Github에서도 preferred_color_mode, timezone 등은 HttpOnly 속성이 체크되지 않은 것을 그림에서 확인할 수 있었습니다.

### Secure

http`s`: 스키마로 요청할 때만 서버(로컬 호스트 제외)로 쿠키가 전송되는 것을 가능하게 하는 설정입니다. 중간자 공격(man-in-the-middle attack)에 더 강하다는 것을 나타냅니다.  
  
> 참고) `Secure` 속성이 쿠키의 민감한 정보(세션 키, 로그인 정보 등)에 대한 모든 액세스를 차단한다고 가정해서는 안 됩니다. 이 속성이 있는 쿠키는 클라이언트의 하드 디스크에 액세스하거나 `HttpOnly` 쿠키 속성이 설정되어 있지 않은 경우 `JavaScript를 통해 여전히 읽기/수정`할 수 있습니다.  
>
> 안전하지 않은 사이트(`http`:)는 보안 속성이 있는 쿠키를 설정할 수 없습니다. Secure 속성이 `로컬호스트에 의해 설정`된 경우 `https: 요구사항은 무시`됩니다.

앞서 Github 의 경우 모든 쿠키에 Secure가 설정된 것을 볼 수 있습니다.

### SameSite

Cross-Site 요청에 대해 쿠키를 보낼 것인지 말 것인지를 결정하는 속성입니다. 그렇다면 Same-Site와 Cross-Site는 무엇을 의미할까요?

#### Site

먼저 Site가 무엇인지 알아보겠습니다. 아래 그림을 보시면, Site는 Scheme과 도메인 네임의 마지막 부분만을 의미하는 것을 볼 수 있습니다.

![origin and domain name](/assets/img/2023-10-22-cookie-attributes/04.site-vs-origin.png)

출처: [PortSwigger Academy](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions)

예제를 살펴보겠습니다.

- `https://example.com`과 `https://example.com`은 앞서 표시된 site 부분이 모두 동일하므로 `Same-Site`가 됩니다.
- `https://app.example.com`과 `https://intranet.example.com`은 `subdomain`이 다르긴 하지만 subdomain은 site에 포함되지 않으므로, `Same-Site`가 됩니다.
- `https://example.com`과 `https://example.com:8080`은 `port`가 다른데, port는 Site에 포함되지 않으므로, `Same-Site`가 됩니다.
- `https://example.com`과 `https://example.co.uk`는 `eTLD(effective Top Level Domain)`가 다르고, 이는 Site 에 포함되므로 `Cross-Site`가 됩니다.
- `https://example.com`과 `http://example.com`는 `scheme`이 다르고, scheme은 Site에 포함되므로 `Cross-Site`가 됩니다.

Same-Site와 Cross-Site는 말 그대로 같은 Site와 다른 Site를 의미한다는 것을 알 수 있습니다.

그럼 Cross-Site에서 요청을 보낼 때 쿠키를 포함시키는게 어떤 문제가 있을까요?

대표적인 예로 (요즘은 모바일에서 모두 해결하지만) 보안이 매우 약한 은행 사이트(bank.com)에서 송금을 한다고 가정해보겠습니다.

은행 사이트에서 송금 작업을 수행하면, 사용자의 브라우저에 session cookie가 남아있게 됩니다. 이때 은행 사이트를 열어둔 채로 악의적인 사이트(evil.com)에 접속했더니, 아래와 같이 img 태그의 src 속성에 설정해둔 해커의 계좌로 송금하는 url이 호출됩니다.

```html
<img src="http://bank.com/transfer?toAccount=attackersAccount&amount=1000" width="1" height="1" />
```

보안적인 조치가 없다면 Cross-Site인 evil.com에서 bank.com을 호출하는데, 이전에 만들어져 있던 session cookie와 함께 요청되어 정상적인 사용자 요청으로 bank.com이 인식하게 됩니다.

이러한 공격을 `CSRF(Cross Site Request Forgery)`라고 하며, 클라이언트에 저장된 정보를 조작하거나 탈취하는 XSS와 달리, 정상 사용자를 가장하여 서버에 있는 정보를 조작하거나 탈취합니다.

그럼 이제 CSRF 공격을 막기위한 방법 중 하나인 SameSite 설정의 종류를 살펴보겠습니다.

#### SameSite 설정의 종류

앞서와 같이 Cross-Site를 허용했을 때 발생하는 문제가 있다보니, 요구되는 보안 수준에 따라 설정이 필요합니다.

총 3가지가 `Strict`, `Lax`, `None` 을 의미하며, 한글로 보자면 `제한`, `완화`, `없음` 정도가 되겠습니다.

##### Strict

```text
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
```

정확하게 Site가 동일한 경우에만 쿠키를 서버로 전송합니다.

##### Lax

```text
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
```

이미지 또는 프레임 로드 요청과 같은 Cross-Site 요청에는 쿠키가 전송되지 않지만 사용자가 외부 사이트에서 origin 사이트로 이동할 때(예: 링크를 따라갈 때) 쿠키가 전송됨을 의미합니다. 이는 SameSite 속성이 `지정되지 않은 경우 기본 동작`입니다.

하지만 기본값 설정은 Chrome도 2021년에서야 설정했고, MDN 문서를 보면 현재 Firefox와 Safari는 기본값을 지원하지 않습니다.

##### None

```text
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=None; Secure
```

None은 브라우저가 Cross-Site 및 Same-Site 요청 시 모두 쿠키를 전송한다는 의미입니다. 이 값을 설정할 때는 Secure 속성도 설정해야 합니다. Secure 속성이 누락되면 오류가 기록됩니다:

> Cookie "myCookie" rejected because it has the "SameSite=None" attribute but is missing the "secure" attribute.
>
> This Set-Cookie was blocked because it had the "SameSite=None" attribute but did not have the "Secure" attribute, which is required in order to use "SameSite=None".

http 사용 시 Secure 속성을 사용할 수 없으므로, 이때는 SameSite를 None으로 설정할 수 없습니다.

#### Partitioned

```text
Set-Cookie: __Host-name=value; Secure; Path=/; SameSite=None; Partitioned;
```

`Secure`와 `Path=/` 는 반드시 설정해야하고, `__Host-` prefix는 권장사항 입니다.

`SameSite=None`을 추가하면 브라우저 설정에서 타사 쿠키(third-party cookie)가 허용되는 한 `partitioned` 속성이 지원되지 않는 third-party context에서 쿠키를 전송할 수 있습니다.

그리고 아직 실험적인 기능입니다. Chrome에서도 작년 즈음 지원하기 시작했습니다.

Partitioned 를 설정하면, `쿠키를 격리된 저장소에 저장`합니다. 왜 그럴까요?

쿠키를 이용한 사용자 추적 등을 제한하여 `사용자 개인정보를 보호`하기 위함입니다.

Partitioned 속성은 별도로 자세히 살펴보는게 좋겠다고 판단되어 이렇게 간단하게 마무리하겠습니다. Chrome for Developers에 상세히 소개된 내용([링크](https://developer.chrome.com/docs/privacy-sandbox/chips/))가 있습니다.

#### Priority

MDN 문서에는 없는데, Chrome 개발자 도구에는 있어서 살펴보니 deprecated 된 속성입니다.

- [A Retention Priority Attribute for HTTP Cookies draft-west-cookie-priority-00](https://datatracker.ietf.org/doc/html/draft-west-cookie-priority-00)
- [Chrome for Developers - View, add, edit, and delete cookies](https://developer.chrome.com/docs/devtools/application/cookies/)

## 가장 안전한 쿠키 설정

```text
Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict
```

Partitioned를 제외한 앞선 쿠키 속성과 관련된 내용을 바탕으로 가장 안전한 쿠키 설정을 해보자면 위와같이 설정할 수 있습니다. 

## Outro

쿠키는 간단하지만, 관련된 주제는 보안과 관련되어있어 쉽지가 않은 것 같습니다. 욕심부리다 좌절하지 않게 차근차근 공부해봐야겠습니다.
