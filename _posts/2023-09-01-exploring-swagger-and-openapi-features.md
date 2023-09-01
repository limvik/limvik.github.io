---
layout: post
title: Swagger와 OpenAPI 특징 살펴보기
categories:
- API
- REST
tags:
- REST
- OpenAPI
- Swagger
date: 2023-09-01 13:06 +0900
---
## Intro

REST Docs 관련해서 제가 잘못알고 있는게 있을까 싶어서 조금 더 찾아보다가 아래 API 명세서와 협업을 주제로 한 영상을 보게 됐습니다.

{% include embed/youtube.html id='r03ObslCNlo' %}

저는 아직 경험해보지 못한 API 제작 시 협업 과정에서 발생하는 의사소통 문제들에 대한 이야기라 재미있게 봤습니다. 그리고 예상했던 REST Docs 사용 시 테스트를 만들지 않아서 API 존재 자체를 알 수 없었던 경우가 있다는 내용이 눈에 들어왔습니다.

이걸 보고나니 더더욱 Swagger 를 사용해보는게 좋다는 생각이 들었습니다.

OAS(OpenAPI Specification) 를 사용하면 `양식이 표준화`되고, codegen 으로 API 코드를 생성하면 `문서 수정이 코드의 변경`이 되므로, 동기화된 OAS 문서를 공유할 방법을 찾는다면 영상에서 나온 의사소통 문제를 많이 해결해줄 수 있지 않을까 생각됩니다.

>그런데 codegen 이 아직 OAS 최신 버전인 3.1을 지원하지 않습니다.

또 OAS 문서는 프론트엔드 개발자도 Postman 에 바로 import 해서 mock server를 만들고 사용할 수 있기도 하니 서로에게 편할 것 같습니다.

Swagger Editor 에 있는 Pet Store 예제를 적당히 수정해서 만들어보려고 했는데, 나름 진지하게 공부해볼만 한 것 같습니다.

manning 출판사에서 나온 [designing apis with swagger and openapi](https://www.manning.com/books/designing-apis-with-swagger-and-openapi) 라는 책이 있어 책을 따라 진행해보려 합니다. 저자 분이 Swagger 개발에 참여했던 분이라고 하네요. 또 영어와의 싸움이  되겠습니다. 든든한 DeepL이 있어 다행입니다.

## Swagger

책에서 나오는 Swagger 에 대해 기록하고 싶은 이야기가 있어서 정리합니다.

### Swagger 라는 이름의 탄생

저는 이런 뒷 얘기를 좋아합니다.

Swagger 이전에 HTTP-based API에 사용하는 `WADL(Web Application Description Language)`이 있었고, `waddle` 이라고 발음합니다. waddle 의 뜻을 사전([링크](https://en.dict.naver.com/#/entry/enko/00ae8d4a98a14c45b0fca25f372f5803))에서 찾아보면, `(오리처럼) 뒤뚱뒤뚱 걷다`라는 뜻을 가지고 있습니다. Swagger 개발 팀이 농담으로 “Why WADL when you can Swagger?” 라고 하다가 Swagger 라는 이름으로 짓게 되었다고 합니다.

Swagger 의 사전 뜻([링크](https://en.dict.naver.com/#/entry/enko/854148e253c84756bbc6ccbce215a350))을 보면 `으스대며[뻐기며] 걷다[활보하다]` 라는 뜻이 있습니다.

Swagger를 쓰면 되지 뭐하러 WADL을 쓰냐는 말을 멋있게 걸을 수 있는데 왜 뒤뚱뒤뚱 걷냐는 농담으로 재밌게 표현했습니다.

Wikipedia([링크](https://en.wikipedia.org/wiki/Web_Application_Description_Language))에서 WADL을 보니, Swagger가 더 간결하고 좋아보입니다.

```xml
 <application xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://wadl.dev.java.net/2009/02 wadl.xsd" 
  xmlns:tns="urn:yahoo:yn" xmlns:yn="urn:yahoo:yn" xmlns:ya="urn:yahoo:api"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  xmlns="http://wadl.dev.java.net/2009/02"> 
   <grammars> 
     <include href="NewsSearchResponse.xsd"/> 
     <include href="Error.xsd"/> 
   </grammars> 
 
   <resources base="http://api.search.yahoo.com/NewsSearchService/V1/"> 
     <resource path="newsSearch"> 
       <method name="GET" id="search"> 
         <request> 
           <param name="appid" type="xsd:string" style="query" required="true"/> 
           <param name="query" type="xsd:string" style="query" required="true"/> 
           <param name="type" style="query" default="all"> 
             <option value="all"/> 
             <option value="any"/> 
             <option value="phrase"/> 
           </param> 
           <param name="results" style="query" type="xsd:int" default="10"/> 
           <param name="start" style="query" type="xsd:int" default="1"/> 
           <param name="sort" style="query" default="rank"> 
             <option value="rank"/> 
             <option value="date"/> 
           </param> 
           <param name="language" style="query" type="xsd:string"/> 
         </request> 
         <response status="200"> 
           <representation mediaType="application/xml" element="yn:ResultSet"/> 
         </response> 
         <response status="400"> 
           <representation mediaType="application/xml" element="ya:Error"/> 
         </response> 
       </method> 
     </resource> 
   </resources>
 </application>
```

2010년도에 REST 표준 문서로 WADL을 사용하길 바라시는 분의 글([링크](https://m.blog.naver.com/shin7688/120105789230))이 있는걸 보니 REST 에 대한 표준 양식의 필요성을 느끼시는 분들이 있었나 봅니다.

### Swagger 와 OAS 용어 구분

OAS와 Swagger는 동의어로 사용되기도 하지만, 정확히는 `Swagger` 는 SmartBear에서 제공하는 툴 셋을 의미하고, `OAS`는 Spec을 의미한다고 합니다.

이는 기존 Swagger에서 Specification 부분만 Linux 재단으로 넘겼는데, 이때 이름이 OpenAPI Specification으로 변경됐고, SmartBear 에서는 Swagger의 상표권을 유지하고 있기 때문입니다.

## OAS

### OAS의 한계

OAS는 HTTP-based API 특히 `RESTful API 를 지원하는 도구`입니다. gRPC, GraphQL 등을 지원하지는 않습니다.

### OAS 버전

OAS 3.1 버전을 제대로 지원하는 곳이 많이 없어서, 책에서는 3.0 버전으로 진행합니다. 저자에 따르면 사용하는데는 큰 차이가 없다고 합니다. 3.0 에서는 JSON Schema 최신 표준(Draft 2020-12)이 아닌 과거 버전(Draft 04)을 따르고, 또 OAS만의 추가적인 변형이 있다고 합니다.

### JSON Schema

JSON Schema는 처음 들어봐서 관련 사이트를 찾아보니 이런 설명이 되어있습니다.

> **JSON Schema** is a declarative language that allows you to **annotate** and **validate** JSON documents.  
> JSON 스키마는 JSON 문서에 `주석`을 달고 `유효성`을 검사할 수 있는 선언적 언어입니다.  
> 
> JSON Schema 2020-12 is a JSON media type for defining the structure of JSON data. JSON Schema is intended to **define validation**, **documentation**, **hyperlink navigation**, and **interaction control** of JSON data.  
> JSON 스키마 2020-12는 JSON 데이터의 구조를 정의하기 위한 JSON 미디어 유형입니다. JSON 스키마는 JSON 데이터의 `유효성 검사`, `문서화`, `하이퍼링크 탐색` 및 `상호 작용 제어`를 정의하기 위한 것입니다.

[https://json-schema.org/](https://json-schema.org/)  
[https://www.learnjsonschema.com/2020-12/](https://www.learnjsonschema.com/2020-12/) 

책에서는 JSON Schema는 JSON으로 할 수 있는 것과 없는 것을 나타내는 방법이라고 깔끔하게 정리해줍니다.

책의 예제를 보면, schema 항목 밑으로 유효성 검사를 위한 설정값들이 보입니다. JSON Schema라는 것을 또 처음 알게됩니다.

```yaml
responses: 
  200:
    description: A bunch of reviews
    content:
      application/json:
        schema:
          type: array
          items:
            type: object
            properties:
              uuid:
                type: string
                pattern: '^[0-9a-fA-F\-]{36}$'
              message:
                type: string
              rating:
                type: integer
                minimum: 1
                maximum: 5
              userId:
                type: string
                pattern: '^[0-9a-fA-F\-]{36}$'
                nullable: true
```

책에 따르면 codegen 을 위해 JSON Schema 를 조금 변형해서 사용하는데, 표준을 따르지 않는 표준이라니 좀 아이러니하기는 합니다.

#### 추가적인 변형 예  

JSON Schema 에서는 가능하지만 OAS에서는 불가능한 표현  

> type: [ number, string, null ]  

위와 같이 JSON Schema 는 여러 type을 지정 가능하지만, OAS에서는 불가능합니다.

Swagger Editor로 직접 해보니, OAS 3.1 버전에서도 지원하지는 않습니다.

![Swagger Editor에서 지정해본 multiple type에 대한 오류 메시지](/assets/img/2023-09-01-Exploring Swagger-and-OpenAPI-features/01.swagger-editor-multiple-type.png)

OAS 3.1에서 JSON Schema 최신 표준을 따르긴 하지만, 일부 변형은 그대로 유지되는 것 같습니다.

참고로 앞의 예제에서 처럼 nullable 정의를 통해 null과 함께 정의하는 것은 가능합니다.

```yaml
userId:
  type: string
  pattern: '^[0-9a-fA-F\-]{36}$'
  nullable: true
```

## Summary

- Swagger는 WADL의 불편함을 해소
- 용어를 정확히 하자면, OAS는 말 그대로 Specification 만을 의미하고, Swagger는 OAS를 사용하기 위한 툴 셋을 의미
- OAS는 유효성 검사를 위해 JSON Schema를 호환하지만, 100% 호환되지는 않음
- OAS 3.1 버전은 3.0 버전 대비 JSON Schema 최신 버전을 호환
- 아직 OAS 최신 버전인 3.1 버전을 지원하는 Swagger 툴이 많이 없음

## Outro

글을 쓸 때는 책을 따라 세부적인 것도 하나하나 정리해볼까 하다가 좋은 자료들이 많이 있는데, 굳이 그럴필요는 없을 것 같아서 간단하게 살펴보는 글로 마무리했습니다.

해외에서는 OAS를 잘 사용하는지 궁금해서 찾아보니, 어느 해외 스타트업 CTO가 쓴 Architecture of modern startup 이라는 글([링크](https://betterprogramming.pub/architecture-of-modern-startup-abaec235c2eb))을 보면, 그림에 OAS 가 보입니다.

![링크에 언급된 Architecture 구조도](/assets/img/2023-09-01-Exploring Swagger-and-OpenAPI-features/02.modern-startup-tech-stack.png)

아직 이해 못할 것들이 많아보여서 제대로 읽어보진 않았지만, 반응을 보니 이제 막 시작하는 스타트업에게는 과한 아키텍처지만, 꽤 성장한 스타트업에게는 괜찮은 아키텍처라는 반응입니다. 본문에서도 OAS를 특별히 언급하지는 않지만, OAS의 특징 중 일부인 API 설계 우선 및 유효성 검사를 언급하고 있습니다.

대체적으로 OAS는 그냥 당연한거라는 느낌이 듭니다. OAS로 아직 개발을 시작하지 않아서 OAS가 좋게만 보이기도 하는 것 같습니다. 작더라도 이제 개발을 시작해 봐야겠습니다.
