---
layout: post
title: Spring REST Docs를 사용하지 않기로 결정한 개인적인 이유
categories:
- Framework
- Spring
tags:
- Spring
date: 2023-11-26 23:43 +0900
---
## Intro

최근에 수행한 개인 프로젝트([링크](https://github.com/limvik/budget-management-service))에서는 Spring REST Docs 를 이용해서 문서화를 해봤습니다.

이전에 Swagger랑 Spring REST Docs를 비교해놓은 글을 보면서 나름대로 비교를 해봐서 Spring REST Docs는 '이런식으로 개발이 되겠구나'라는 생각을 갖고 있었는데, 생각과는 다른 부분들이 조금 있어서 고생을 했습니다.

사용해보면서 사용 전에 잘못 알고있던 것들을 정리하고, 저는 Spring REST Docs가 Swagger 대비 별로였어서, 다음부터는 개인 프로젝트를 한다면 Spring REST Docs를 사용하지는 않기로 결정했는데 그 이유에 대해 정리해보겠습니다. `Spring REST Docs를 처음 사용하시려는 분`들에게 도움이 되었으면 합니다.

## Spring REST Docs 에 대해 잘못 알고있던 것들

### 1. 문서를 자동으로 만들어줄 것이다.

첫 테스트를 작성한 후 기대감을 갖고 build 했는데, 문서가 안만들어집니다.

기본 경로인 `build/generated-snippets` 에는 분명 snippets 가 있는데, 이를 통합한 문서가 안만들어집니다. 왜지...? 했는데, 원래 snippets 만 만들어주고, snippets 를 조합해서 직접 문서를 꾸며줘야 합니다.

snippets 조합은 어디서 하는가? `src/docs/asciidoc` 경로에 `index.adoc` 파일을 직접 만들어주어야 합니다.

adoc 문서를 만들려면 [`asciidocs`](https://docs.asciidoctor.org/) 도 새롭게 배워야하고, Custom Snippet를 만드려면 [`mustache`](https://mustache.github.io/) 도 배울 필요가 있습니다.

제가 사용했던 Custom Mustache Template을 간단하게 보면 아래와 같습니다.

![Figure.00 Mustache](/assets/img/2023-11-26-review-spring-rest-docs/00.mustache.png)

Request Fields의 기본 항목인 Path, Type, Description 이외에 별도로 추가한 Constraints는 아래와 같이 테스트 작성 시 `attributes` 메서드에 지정하여  사용할 수 있습니다.

```java
    private RequestFieldsSnippet getExpenseRequestFields() {
        return requestFields(
                fieldWithPath("datetime")
                        .type(JsonFieldType.STRING)
                        .description("지출 일시")
                        .attributes(key("constraints").value("Nullable")),
```

잠깐보면 이해하고 쓸 수 있을 정도로 간단하기는 합니다. 물론 정말 잘 쓰는 것은 시간 투자가 조금 필요해 보입니다.

### 2. Spring Project니까 문서가 굉장히 잘 되어있을 것이다.

Spring 을 사용하면서 문서에 대한 경험이 워낙 좋아서 Spring REST Docs도 당연히 문서가 친절하게 잘 돼있을거라 생각했습니다.

특히 제가 무엇을 잘못했는지, 공식 문서에 나온 내용대로 따라해서는 제가 작성한 API 문서를 볼 수가 없었습니다.

공식 문서에 나온 bootJar 작업에 대한 설정은 아래와 같습니다.

```groovy
bootJar {
	dependsOn asciidoctor 
	from ("${asciidoctor.outputDir}/html5") { 
		into 'static/docs'
	}
}
```

그리고 다른 분의 블로그([링크](https://gaemi606.tistory.com/entry/Spring-Boot-REST-Docs-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0))를 따라 html5 경로를 제거하여 문서를 볼 수 있었습니다.

```groovy
bootJar {
	dependsOn asciidoctor
	from ("${asciidoctor.outputDir}") {
		into 'static/docs'
	}
}
```

실제 경로도 `build/docs/asciidoc` 에 `index.html` 이 생성되어 있는 것을 볼 수 있습니다.

![Figure01. build/docs/asciidoc 프로젝트 경로 화면](/assets/img/2023-11-26-review-spring-rest-docs/01.path-index-html.png)

### 3. Spring REST Docs 는 문서에서 직접 API 요청을 보낼 수 없다.

Swagger 대비 Spring REST Docs의 대표적인 단점으로 항상 등장하는 단점입니다.

그런데 asciidocs 문법을 기반으로 HTML 문서를 만드는데 `HTML하고 JavaScript가 안될까?` 하는 의문이 생겨 찾아보니 [`passthrough block`](https://docs.asciidoctor.org/asciidoc/latest/pass/pass-block/)을 통해 직접 HTML과 JavaScript 를 사용할 수 있었습니다.

![Figure02. Spring REST Docs 에서 HTTP 요청 보내고 응답 표시한 화면](/assets/img/2023-11-26-review-spring-rest-docs/02.http-request-in-spring-rest-docs.png)

```html
[.text]
++++
<button id="requestButton" value="요청 보내기">Request</button>

<textarea id="requestBody"></textarea>
<div id="apiResponse"></div>
++++

[.javascript]
++++
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script>
$(document).ready(() => {
    $('#requestButton').click(() => {
        request("http://localhost:9001/api/v1/users/signin", "POST", null, $('#requestBody').val());
    });

    function request(url, method, headers, body) {
        console.log(body);
        $.ajax({
            url: url,
            type: method,
            dataType: 'json',
            data: body,
            contentType: 'application/json',
            headers: headers,
            success: function (xhr, status, response) {
                $('#apiResponse').text(response.responseText);
            },
            error: function (xhr, status, error) {

            }
        });
    }
})
</script>
++++
```

이런식으로 Spring REST Docs 를 사용하면서 HTTP 요청이 안되는건 아니라는 것을 확인할 수 있었습니다.

## Spring REST Docs를 사용하지 않기로 결정한 이유

전체적인 이유는 Swagger 대비 Spring REST Docs의 장점을 찾기 어려웠기 때문입니다.

먼저 제가 Swagger를 사용하는 방식은, 보통의 블로그 글에서 살펴볼 수 있는 Swagger 사용 방식인 Java 코드에 annotation을 직접 타이핑하여 Swagger 문서를 만드는 방식과는 조금 다름을 말씀 드려야 할 것 같습니다.

저는 API 설계하면서 [Swagger Editor](https://editor-next.swagger.io/)를 이용해서 `OpenAPI Specification 문서를 먼저 작성`합니다. 이후에는 선택지가 있습니다. Postman에 import 해서 문서를 생성해도 되고, Code Generator를 이용해서 Java 코드를 자동 생성해도 됩니다. 자동 생성한 코드가 찝찝하면 자동 생성한 코드에서 필요한 부분만 복사해서 사용합니다.

그럼 Spring REST Docs를 사용하지 않기로 결정한 구체적인 이유를 나열해 보겠습니다. 아직 현업 경험이 없는 개발자 지망생의 굉장히 주관적인 의견이므로 적당히 걸러 들으시면 됩니다.

### 1. 테스트 작성을 강제할 뿐 테스트 퀄리티를 강제하지는 않는다.

제가 봤던 블로그 글에 한해서 테스트를 강제한다는 것이 Spring REST Docs를 사용하는 가장 큰 이유이고, 제가 사용해보고 싶었던 이유이기도 합니다. 테스트를 통과하면 검증된 API가 될 수 있기 때문입니다.

그런데 검증된 API를 이야기 하면서 `MockMvc`를 사용하는지 납득이 안됩니다. `MockMvc` 를 이용해서 슬라이스 테스트로 Controller만 테스트하고 검증된 API의 문서라고 주장하기는 어렵다는 것이 제 생각입니다. 그렇다고 `RestAssured`로 테스트를 작성한다고, 무조건 검증된 API가 되는 것도 아닙니다. 테스트도 작성하기 나름이기 때문입니다.

또한 테스트를 작성하기 귀찮아서 Shadow API(실제로 존재하지만, 문서상에는 없는 API)가 되는 사례도 찾아볼 수 있는 것으로 보아 단순히 테스트를 작성을 강제한다는 것이 그다지 큰 메리트가 있어보이지는 않습니다.

### 2. 문서가 자동으로 생성되지 않는다.

자동 생성된 Snippets 를 가지고 자유롭게 문서를 구성할 수 있어 장점이 될 수도 있지만, 저에게는 추가적인 시간을 투자해야하는 단점으로 다가왔습니다.

예를들어, Spring REST Docs는 Request Fields 나 Response Fields 와 같이 정리해서 보여줘야 할 것들을 `표(table)`로 정리해준다는 장점이 있습니다. 개인적으로는 문서의 Look & Feel도 Swagger 보다 좋습니다. 그렇다고 Swagger의 가독성과 시각적인 부분이 완전 바닥이거나 한 것은 아닌데, 문서를 구성하기 위해서 추가적인 시간을 투자해야 한다는 것이 개인적으로는 굉장히 비효율적으로 느껴졌습니다.

---

정리해 보자면 주관적으로 봤을 때, Spring REST Docs에서 `테스트를 강제하는 정도가 검증된 API를 만드는 정도는 아니`라고 생각되며, `문서를 직접 구성하는 노력을 들여야할 만큼의 가독성 향상은 없다`고 생각됩니다. Spring REST Docs의 장점이라 생각했던 것들이 모두 기대 이하라는 생각이들었고, 개인 프로젝트에서는 사용하지 않기로 결정하였습니다.

테스트만 작성하면 이쁜 API 문서를 얻을 수 있다는 저의 상상이 와르르...

### Swagger와 Spring REST Docs 둘 다 처음이라면?

만약 Swagger와 Spring REST Docs가 둘 다 처음이라면, 조금 고민이 많이 될 것 같습니다.

Swagger 사용 시에 OpenAPI Specification 문서를 작성하는 방법을 익히는데 꽤 시간이 걸리기 때문입니다. 처음에 다른 개인 프로젝트에서 Swagger 사용하기로 결정했을 때 OpenAPI Specification 문서 작성법 보다가 살짝 후회하기도 했었습니다. (아직도 잘은 못쓰는게 문제...)

결국 둘 다 처음이라면 취향의 차이로 결정되지 않을까 싶습니다. 생각나는 대로 적어보자면 아래와 같습니다.

- Spring REST Docs
  - 상대적으로 더 나아보이는 Look & Feel 
  - 목차 및 표를 이용한 가독성 및 사용성 향상
  - 자유로운 API 문서 구성 가능
- Swagger
  - 정해진 양식으로 API 문서 자동 생성
  - 표준 OAS(OpenAPI Specification) 문서를 협업 시 API 계약 문서로 활용 가능
  - Postman과 같은 다른 Tool을 통해 OAS 문서 활용 가능
  - 다른 언어를 사용 시에도 재활용 가능

개인적으로 Swagger가 더 낫다고 생각해서 뭔가 조금 더 좋은 어감으로 작성된 것 같습니다.

## Outro

테스트 코드 작성하는 것을 선호해서 꽤나 기대감을 갖고 Spring REST Docs를 사용해봤는데, 실망해서 단점 위주로 작성하게 된 것 같습니다.

혹시나 오해를 하시게 되실까봐 마지막으로 변명을 좀 하자면, Spring REST Docs 가 나쁜 것이 아니라 Swagger를 사용해본 경험 대비 별로 좋은 경험을 하지 못해서 사용하지 않는 것입니다.

Swagger도 OpenAPI Specification 문서를 작성해야 한다는 꽤나 큰 장애물이 있기 때문에, 상황에 맞게 잘 고려해서 선택해야 합니다.
