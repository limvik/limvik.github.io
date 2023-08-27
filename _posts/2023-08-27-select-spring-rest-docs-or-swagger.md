---
layout: post
title: Spring REST Docs? Swagger? 무엇을 선택해야할까
categories:
- API
- REST
tags:
- Spring
- Swagger
date: 2023-08-27 16:18 +0900
---
## Intro

얼마전 원티드 프리온보딩 백엔드 과제로 REST API를 만들어봤었습니다. 제출한 서류도 과제도 지원한 사람이 많다는 이유로 확인도 안하고 탈락시켜서 마음의 상처를 받았습니다... 어쨌거나 그래도 처음 REST API를 만들어보면서 많은 것을 배울 수 있었습니다.

그런데 당시 시간이 부족해서 API 문서는 Postman을 이용해서 수작업으로 만들었습니다.

![Postman 으로 작성한 API 문서 화면](/assets/img/2023-08-27-select-spring-rest-docs-or-swagger/01-postman-api-docs.png)

Postman은 무료로 API 문서 호스팅도 해줍니다(언제 지워질지 모를 [링크](https://documenter.getpostman.com/view/4269053/2s9Xy6p9qJ)).

REST API도 API 문서도 처음 만들어보니 허접하긴 하지만, Postman의 도움으로 만들어볼 수 있기는 했습니다. 그래도 API 문서 만들 때 매번 수작업 할 수는 없으니, 방법이 필요했습니다.

이전에 책을 보면서 Swagger 로 프로젝트 생성하는데 문제가 있어 글([링크](/posts/how-to-use-swagger-generator-6-6-0-in-spring-boot-3-x/))을 작성하기도 하고 한 번 접해보기는 했습니다. 그런데 전반적인 책 내용이 제 수준에 비해 높아서 프로젝트 생성만 하고 중단했었습니다. 이제는 본격적으로 공부해볼까 생각하다가 YAML이나 JSON 파일 만들기가 귀찮아서 다른 방법은 없나 찾아보다가 Spring REST Docs를 이용한 방법을 발견했습니다.

Spring REST Docs를 이용한 방법도 좋은 선택으로 보였고, Spring REST Docs 와 Swagger를 모두 이용하는 분도 계셨지만, 저는 다음에 만들어볼 REST API에서 `Swagger`를 이용하기로 결정했습니다. 결정 과정에서 조사했던 자료들을 정리할 겸 글을 작성해봅니다.

## 선택 기준 찾아보기

여러가지 블로그 글을 찾아봤는데, 비교한 내용은 대동소이 해서 아래 두 자료를 저장 겸 추가해둡니다.

[내가 만든 API를 널리 알리기 - Spring REST Docs 가이드편](https://helloworld.kurly.com/blog/spring-rest-docs-guide/)
[Swagger와 Spring Restdocs의 우아한 조합 (by OpenAPI Spec)](https://taetaetae.github.io/posts/a-combination-of-swagger-and-spring-restdocs/)  
[우아한 기술 블로그 - Spring Rest Docs 적용](https://techblog.woowahan.com/2597/)  

특성을 대략 표로 정리해보면, 아래와 같습니다.

|  | 프로덕션 코드 영향 | 테스트 강제 | 문서 내 API 호출 |
|--|--|--|--|
| Spring REST Docs | X | O | X |
| Swagger | O | X | O |

자료에 나온 장점과 단점도 살펴보면, 아래와 같습니다.

|  | 장점 | 단점 |
|--|--|--|
| Spring REST Docs | 테스트 강제로 안정성 높음 | 상대적으로 설정 어려움 |
|  | 프로덕션 코드 영향 X | 테스트 코드가 길어짐 |
| Swagger | API 현행화 쉬움 | 검증되지 않은 API 존재 가능  |
|  | API 문서에서 호출 가능 | 프로덕션 코드 영향 O |

우아한 기술 블로그에서는 다음과 같은 이유로 최종 선택을 내렸습니다.

>API 문서의 목적은 개발하는 스펙을 정의하는것이라 생각합니다.  
>Swagger는 API 동작을 테스트하는 용도에 더 특화되어있습니다.  
>반면에 Spring Rest Docs 은 깔끔 명료한 문서를 만들수 있습니다.  
>그래서 문서 제공용으로는 Spring Rest Docs 이 좀 더 낫다고 생각합니다.  

선택 기준을 정리해보면, 대략 이정도로 정리할 수 있겠습니다.

1. 둘 다 사용
2. 문서의 깔끔 명료성
3. 테스트 강제성
4. 프로덕션 코드 영향 여부
5. API 문서에서 호출 가능 여부
6. 설정 난이도

## 선택 기준 검토해보기

### 둘 다 사용하기

`둘 다 쓰시는 분을 따라하기`에는 아직 한 가지라도 제대로 사용해보는게 낫다고 판단돼서 `패스`.

### 문서의 깔끔 명료성

우아한 기술 블로그에 나온 것 처럼 `깔끔 명료한 문서 여부`를 기준으로 삼을 것인가 고민해봤지만, Spring REST Docs가 더 깔끔 명료한 문서를 만들 수 있다는 것은 `주관적인 개인 취향`이라, Swagger 문서 스타일을 더 좋아하는 저에게는 선택의 기준이 될 수가 없으므로 `패스`.

### 테스트 강제성

그렇다면 `테스트 강제성`을 기준으로? Spring REST Docs 에서는 API가 존재하더라도 테스트를 작성하지 않으면 문서에서는 존재하지 않는 API가 되고, Swagger 에서는 테스트를 작성하지 않아도 문서에 API 존재가 나타나지만 안정성이 떨어집니다. 어떻든 양쪽 다 API가 존재할 수는 있습니다. 그렇다면 개인적으로는 테스트가 없더라도 API를 구현했었는지 알 수 있는게 낫지 않을까하는 생각이 들어 일단 `Swagger 1승` 입니다.

그렇다면 엄청난 안정성을 요하는 경우 테스트가 없다면 쓰지도 말라는 의미에서 Spring REST Docs 를 쓰는게 맞나? 싶지만, 존재 자체를 잊어버린다면 보안 구멍이 될 수 있을 것 같아 판단하기가 어렵습니다.

참고로, 유료이기는 하지만 Swagger 를 support 하는 SMARTBEAR 에서 제공하는 테스트 툴([링크](https://smartbear.com/product/ready-api/multiprotocol))도 있기는 합니다.

어떻게든 테스트 해보려는 노력들도 유튜브에 보입니다.

[How to fast generate your API Test with OpenAPI Tools and Rest-Assured by Elias Nogueira](https://www.youtube.com/watch?v=uIzb6QiGXsE)  
[Dynamic Test Generation with OpenAPI 3.0, Allen Helton, Tyler Tech | Postman Galaxy 2021](https://www.youtube.com/watch?v=fzN67jPLpqI)  

여튼 테스트를 작성하지 않아서 만들어놓은 API 존재 자체를 잊는 것 보다는 다른 방식으로 보완하는게 더 나은 것 같아 Swagger에게 1승을 주었습니다.

### 프로덕션 코드 영향 여부

Swagger는 프로덕션 코드 영향이 있다는 점을 단점으로 꼽는 이유는 추측이긴 하지만, 내가 눈으로 보고 수정해야 하는 코드에 Swagger 문서화용 Annotation 들이 덕지덕지 붙어야하기 때문이라 생각됩니다. 그런데 제가 드는 의문은 왜 구현 클래스에 문서화용 Annotation 을 덕지덕지 사용하는가 입니다.

Swagger Editor의 Pet Store 예제([링크](https://editor-next.swagger.io/?_ga=2.110646274.605272881.1693105610-1905464902.1689296239))를 이용해 Spring 코드를 자동생성해서 다운로드 받아보면, Interface를 생성하여 Annotation이 작성됩니다.

일부만 살펴보면 아래와 같습니다.

```java
@javax.annotation.Generated(value = "io.swagger.codegen.v3.generators.java.SpringCodegen", date = "2023-08-27T03:13:36.668360388Z[GMT]")
@Validated
public interface PetApi {

    @Operation(summary = "Add a new pet to the store", description = "Add a new pet to the store", security = {
        @SecurityRequirement(name = "petstore_auth", scopes = {
            "write:pets",
"read:pets"        })    }, tags={ "pet" })
    @ApiResponses(value = { 
        @ApiResponse(responseCode = "200", description = "Successful operation", content = @Content(mediaType = "application/json", schema = @Schema(implementation = Pet.class))),
        
        @ApiResponse(responseCode = "405", description = "Invalid input") })
    @RequestMapping(value = "/pet",
        produces = { "application/json", "application/xml" }, 
        consumes = { "application/json", "application/xml", "application/x-www-form-urlencoded" }, 
        method = RequestMethod.POST)
    ResponseEntity<Pet> addPet(@Parameter(in = ParameterIn.DEFAULT, description = "Create a new pet in the store", required=true, schema=@Schema()) @Valid @RequestBody Pet body);
```

그리고 자동 생성된 Controller 를 일부 살펴보면, Interface를 구현하여 문서화를 위한 Annotation을 볼 일은 없습니다.

```java
@javax.annotation.Generated(value = "io.swagger.codegen.v3.generators.java.SpringCodegen", date = "2023-08-27T03:13:36.668360388Z[GMT]")
@RestController
public class PetApiController implements PetApi {

    private static final Logger log = LoggerFactory.getLogger(PetApiController.class);

    private final ObjectMapper objectMapper;

    private final HttpServletRequest request;

    @org.springframework.beans.factory.annotation.Autowired
    public PetApiController(ObjectMapper objectMapper, HttpServletRequest request) {
        this.objectMapper = objectMapper;
        this.request = request;
    }

    public ResponseEntity<Pet> addPet(@Parameter(in = ParameterIn.DEFAULT, description = "Create a new pet in the store", required=true, schema=@Schema()) @Valid @RequestBody Pet body) {
        String accept = request.getHeader("Accept");
        if (accept != null && accept.contains("application/json")) {
            try {
                return new ResponseEntity<Pet>(objectMapper.readValue("{\n  \"photoUrls\" : \"\",\n  \"name\" : \"\",\n  \"id\" : \"\",\n  \"category\" : {\n    \"name\" : \"\",\n    \"id\" : \"\"\n  },\n  \"tags\" : \"\",\n  \"status\" : \"\"\n}", Pet.class), HttpStatus.NOT_IMPLEMENTED);
            } catch (IOException e) {
                log.error("Couldn't serialize response for content type application/json", e);
                return new ResponseEntity<Pet>(HttpStatus.INTERNAL_SERVER_ERROR);
            }
        }

        return new ResponseEntity<Pet>(HttpStatus.NOT_IMPLEMENTED);
    }
```

Code Generator 로 생성한 코드이기는 하지만, 직접 작성하더라도 Interface에 작성하면 내가 눈으로 봐야하는 구현 클래스를 더럽히는 일은 없습니다.

그렇다면 Interface 를 작성하기 위한 파일이 늘어나서 복잡해지지 않을까? 생각해보면, 테스트 파일 늘어나는거랑 별 차이가 없을 것 같습니다.

Swagger는 프로덕션 코드에 영향을 주기는 하지만, 직접적으로 코드를 읽는 과정에 영향을 줄 일은 없습니다. 이런 점에서 볼 때, Swagger의 단점이라고 하기도 애매모호해서 선택 기준이 될 수는 없으므로 `패스`.

### API 문서에서 호출 가능 여부

API를 호출할 대체 수단이 많고, Spring REST Docs가 많이 선택되는 것으로 봐서는 중요하게 고려할 기준은 아니라 판단되므로 `패스`.

### 설정 난이도

찾아본 자료에서는 Spring REST Docs 가 상대적으로 설정하기 어렵다고들 하셨는데, 제가 사용해본 경우에는 Swagger 는 훨씬 어려웠습니다. Swagger를 사용하는 경우 OpenAPI Spec 도 공부해야하고, Code Generator를 사용하는 경우에는 추가적인 설정이 필요합니다.

설정 난이도 면에서는 `Spring REST Docs 1승`.

### 추가적인 기준 고려하기

다른 사람 기준으로 좀 쉽게 선택해보려 했더니 쉽지 않습니다. 1:1 이라니...

그렇다면 배운 지식을 기반으로 고려해 볼 때, API의 표준 필요성에 의해 생겨난게 OpenAPI Spec 이라는 점, OpenAPI Spec 에 따라 YAML 또는 JSON 문서로 만들어놓으면 다른 언어에서도 사용 가능하다는 점을 고려하여 Swagger 를 선택하겠습니다.

요약해보자면 API 표준에 맞춰 REST API를 만들어볼 수 있는 Swagger 를 선택했습니다.

특히나 아직 REST API를 한 번 밖에 못 만들어본 상태라, 표준에 맞춰서 만드는 연습하기에도 좋을 거라 생각했습니다.

## Outro

제가 느끼기엔 Swagger는 API 문서화 도구가 아니라, API 를 만드는 도구입니다. MS Azure 의 API design Best Practice 문서([링크](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#open-api-initiative))를 보면, 더욱 그렇게 느껴집니다. 문서에 나와있는 Open API를 도입할 때 고려할 사항 중에 아래와 같은 내용이 있습니다.

> OpenAPI 사양에는 `REST API를 어떻게 설계해야 하는지`에 대한 일련의 의견 지침이 포함되어 있습니다. 상호 운용성에는 이점이 있지만 사양을 준수하도록 API를 설계할 때는 `더 많은 주의가 필요`합니다.
>
> OpenAPI는 구현 우선 접근 방식이 아닌 `계약 우선 접근` 방식을 장려합니다. 컨트랙트 우선이란 API 컨트랙트(인터페이스)를 먼저 설계한 다음 컨트랙트를 구현하는 코드를 작성하는 것을 의미합니다.

API를 설계하는 일도, 객체 지향에서 메시지를 먼저 선택하라는 것과 같은 프로세스를 권장하고 있습니다.

이쯤되면 Swagger 를 문서화 도구인 Spring REST Docs랑 비교하는게 맞나? 싶은 생각도 듭니다.

Swagger 를 선택했으니, REST API 를 표준에 맞게 만들면서 테스트는 어떻게 작성할 수 있을지 고민해 봐야겠습니다. 이러다 취업과는 멀어지는게 아닌가 싶은 불안감이 들기는 합니다.

## Open API 관련 링크

[https://openapi.tools/](https://openapi.tools/)  
[Swagger Editor](https://editor-next.swagger.io/)  
[https://www.openapis.org/](https://www.openapis.org/)
