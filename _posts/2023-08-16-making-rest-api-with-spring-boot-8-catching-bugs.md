---
layout: post
title: Spring Boot로 REST API 만들어보기 (8) 버그 잡기
categories:
- API
- REST
tags:
- Spring
- JUnit5
- Regex
date: 2023-08-16 23:33 +0900
---
## Intro

제출 전에 시간이 좀 남아서, 버그들을 수정하려고 합니다.

## 특정 게시물 조회 시 Query String이 붙은 경우 403(Forbidden) 반환

### 상황

아래와 같이 특정 게시물을 조회하는 URL 을 입력하면 403(Forbidden)이 발생하는 문제가 있습니다.

>http://localhost/api/v1/posts/13?size=3&page=0

```json
{"timestamp":"2023-08-16T11:03:25.839+00:00","status":403,"error":"Forbidden","path":"/api/v1/posts/13"}
```

### 원인

그 이유는 SecurityConfig 에서 정규표현식을 잘못 작성했기 때문입니다.

```java
"/api/v1/posts/\d+"
```

위 정규표현식은 /api/v1/posts/가 앞에 오고, 마지막이 무조건 숫자로만 구성되어야 매칭이 됩니다.

### 정규표현식 수정

정규표현식에 약해서 정규표현식 작성 머신 GPT에게 도움을 청했는데, 썩 결과가 좋지 못해 이것저것 정보를 조합해서 만들어 봅니다.

```java
"^(/api/v1/posts/)(\\d+)(\\?.*)?"
```
^ : 뒤에 오는 (/api/v1/posts/) 으로 시작하게 해줍니다.
\\\\d+ : 숫자만 필터링 합니다.
\\\\?.* : 물음표로 시작하고 뒤에는 아무 문자나 올 수 있습니다.
? : 앞의 (\\\\?.*) 는 있을 수도 있고 없을 수도 있습니다. 

### SecurityConfig 수정

여러 곳에서 사용하므로, 수정하는 김에 여러 곳에서 사용하므로 상수로 만들어버립니다.

```java
private static final String POST_REGEX = "^(/api/v1/posts/)(\\d+)(\\?.*)?";
```

SecurityFilterChain의 matcher 를 수정해줍니다.

```java
auth.requestMatchers(new RegexRequestMatcher(POST_REGEX, "PATCH")).hasRole("USER");
auth.requestMatchers(new RegexRequestMatcher(POST_REGEX, "DELETE")).hasRole("USER");
auth.requestMatchers(new RegexRequestMatcher(POST_REGEX, "GET")).permitAll();
```

### 결과

로그를 확인해보면, 아래와 같이 매칭이 되는 것을 확인할 수 있습니다.

> RegexRequestMatcher - Checking match of request : '/api/v1/posts/13?size=3&page=0'; against '^(/api/v1/posts/)(\d+)(\?.*)?'

마음이 급해서 테스트를 작성하지 않았네요. 그래도 아직은 조금의 여유가 있으니까 테스트를 작성합니다.

### 테스트 작성

기존의 테스트를 ParameterizedTest로 변경하여 다른 url 이 만들어지도록 합니다.

```java
@ParameterizedTest
@DisplayName("존재하는 게시글 ID로 게시글 조회 요청")
@CsvSource({"1, ", "1, ?page=0&size=5"})
void shouldReturnPostIfValidId(String resource, String queryString) {
    postRepository.saveAll(getNewPosts(1));
    
    String url = "/api/v1/posts/" + resource + Optional.ofNullable(queryString).orElse("");
    ResponseEntity<String> createResponse =
            restTemplate.getForEntity(url, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    URI postsList = URI.create(documentContext.read("$.postsList"));
    assertThat(postsList).hasPath("/api/v1/posts");

    Map<String, Object> postsInfo = documentContext.read("$.post");
    assertThat(postsInfo.size()).isEqualTo(6);
    assertThat(postsInfo.get("id")).isEqualTo(1);
    assertThat(postsInfo.get("title")).isEqualTo("title1");
    assertThat(postsInfo.get("content")).isEqualTo("content1");
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get("createdAt")))).isBefore(LocalDateTime.now());
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get("updatedAt")))).isBefore(LocalDateTime.now());

    Map<String, Object> user = (Map<String, Object>) postsInfo.get("user");
    assertThat(user.get("id")).isEqualTo(1);
    assertThat(user.get("email")).isEqualTo("limvik@limvik.com");
    assertThat(user.get("password")).isNull();
}
```

### 추가적인 문제 파악

사실 이 문제를 어제부터 수정 시도했는데 맞다고 생각한 모든 정규표현식이 안돼서, 오늘 갑자기 Docker 에 변경된 소스가 반영이 안되겠다는 생각이 들어서 확인해보니 생각대로였습니다.

소스가 변경되면 Docker 컨테이너에도 자동적으로 반영될 수 있게 수정해야 합니다.

하지만... 그전에 이제 정상적으로 반환되는 값을 보면 아래와 같습니다.

```java
{
    "postsList": "http://localhost/api/v1/posts",
    "post": {
        "id": 13,
        "title": "개발자 포지션 지원시 준비사항",
        "content": "개발자로 지원할 때 필요한 기술 스택과 준비 사항을 알려드립니다.",
        "createdAt": "2023-08-04T14:12:00",
        "updatedAt": "2023-08-04T14:12:00",
        "user": {
            "id": 1,
            "email": "limvik@limvik.com",
            "createdAt": "2023-07-26T10:30:17"
        }
    }
}
```

문제는 `postsList` 입니다. 이렇게 Query String이 포함된 경우를 신경 쓴 이유는 기존 게시판 목록으로 돌아가는 링크를 클라이언트에 제공하기 위함인데, page와 size 정보가 없습니다.

```json
"postsList": "http://localhost/api/v1/posts",
```

제시된 요구사항은 아니지만, 의도를 갖고 작성했던게 뜻대로 안되면 버그이므로 수정합니다.

## 특정 게시물 조회 시 Query String이 붙은 경우에도 반환되는 게시글 목록 URL에 Query String이 없음

상황은 이미 위에서 적었으므로, 원인을 파악해 봅니다.

### 원인

디버거로 UriComponentsBuilder 를 확인해보니 요청이 올때부터 Query 와 관련된 정보가 전혀 없는 것을 알 수 있습니다.

![디버거로 확인한 UriComponentsBuilder 내 QueryParams](/assets/img/2023-08-16-making-rest-api-with-spring-boot-8-catching-bugs/01-no-query-params.png)

Spring에서 알아서 Parsing을 해줄거라고 생각하고 있었는데, 잘못알고 있었던 겁니다.

### 테스트 수정

앞의 특정 게시글 조회 테스트에서 한 가지 테스트 항목을 추가합니다.

```java
if (queryString != null)
            assertThat(postsList.getQuery()).isEqualTo(queryString);
```

### Controller 수정

Controller 에서 Query 를 Map 형태로 받아 Service에 넘겨줄 때 UriComponentsBuilder의 인스턴스에 추가하여 보내줍니다.

```java
@GetMapping("/{id}")
public ResponseEntity<PostResponse> getPost(@PathVariable long id,
                                            @RequestParam MultiValueMap<String, String> queries,
                                            UriComponentsBuilder ucb) {
    PostResponse postResponse = postService.getPostResponse(id, ucb.queryParams(queries));

    if (postResponse != null) {
        return ResponseEntity.ok(postResponse);
    } else {
        return ResponseEntity.notFound().build();
    }
}
```

### 테스트

테스트를 해보면... 

![디버거로 확인한 수정 후 Query String](/assets/img/2023-08-16-making-rest-api-with-spring-boot-8-catching-bugs/02-add-query-params.png)

그리고 이어서 결과를 보면, `?` 를 빼야합니다.

```
expected: "?page=0&size=5"
 but was: "page=0&size=5"
```

물음표를 잘라버립니다.

```java
assertThat(postsList.getQuery()).isEqualTo(queryString.substring(1));
```

그리고 또 테스트를 해보면 PASSED 됩니다.

```
존재하는 게시글 ID로 게시글 조회 요청 > [1] resource=1, queryString=null PASSED
존재하는 게시글 ID로 게시글 조회 요청 > [2] resource=1, queryString=?page=0&size=5 PASSED
```

Postman 으로도 결과를 확인해보면 잘 나옵니다.

![Postman으로 확인한 postList 링크 내 Query String](/assets/img/2023-08-16-making-rest-api-with-spring-boot-8-catching-bugs/03-queries-in-postList.png)

다음으로 앞서 언급했던 소스코드 변경 시 Docker 컨테이너에 실시간 반영이 되게 하려고 했는데, 좀 찾아보다 보니 시간이 너무 잘 가서 마감시간 넘길 것 같아 제출하고 수정을 진행해야겠습니다.

## Outro

이제는 또 다시 새로 시작한 팀 프로젝트에 집중해야 될 때가 됐습니다. 뭔가 되돌아볼 새도 없이 제자리 걸음하는 느낌입니다.
