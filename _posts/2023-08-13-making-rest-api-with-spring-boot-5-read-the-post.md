---
layout: post
title: Spring Boot로 REST API 만들어보기 (5) 특정 게시글 조회
categories:
- API
- REST
tags:
- Spring
- Spring Security
- JPA
- JUnit5
date: 2023-08-13 20:53 +0900
---
## Intro

게시글 목록까지 받아왔고([링크](/posts/making-rest-api-with-spring-boot-4-request-posts-list/)), 다음 과제로 넘어갑니다.

## 요구사항

### 과제 5. 특정 게시글을 조회하는 엔드포인트

- 게시글의 ID를 받아 해당 게시글을 조회하는 엔드포인트를 구현해 주세요.

게시글의 ID를 받을 방법은 게시물 생성 시에 Location 헤더를 통해 받거나, 게시글 목록 조회를 통해서 게시글의 ID를 받을 수 있습니다.

이번에는 뭔가 간단할거 같은... 부질없는 상상을 해봅니다.

## 특정 게시글 조회 엔드포인트

계속 사용해온 경로인 `/posts`에 id가 하나 더 붙습니다.

### 경로

> /posts/{id}

### HTTP method

게시글의 id가 비밀도 아니고, id 외에 body를 통해서 특별히 다른 정보를 보내야 하는 것도 아니니 `POST` 를 통해서 가져와야 할 이유는 없습니다. 데이터를 가져오는 `GET` 을 사용합니다.

> GET /posts/{id}

### HTTP 요청 성공 시 response status code

성공 시에는 게시글 목록 조회했을 때와 동일하게 `200(OK)`으로 응답합니다. 해당하는 id가 없는 경우에는 `404(Not Found)` 로 응답합니다.

### HTTP 요청 실패 시 response status code

id 자리에 문자가 들어간 경우에는 추후에 다른 경로가 생길 수도 있으니 Spring Security 가 알아서 응답하게 `403(Forbidden)`으로 놔둡니다.

### 응답 양식

기본적으로 `게시글 내용`이 있어야겠고, `게시글 목록으로 돌아갈 수 있는 링크`를 하나 추가해주겠습니다.

## 테스트 및 구현 목록

이전에 그냥 생각나는대로 테스트하고 구현했더니 혼란스러운 것 같아서 간단하게라도 목록을 정리하고 가야겠습니다. 라고 쓰고 목록을 정리했는데, 몇개 없습니다.

- 존재하는 게시글 ID 게시글 조회 요청한 경우
- 존재하지 않는 게시글 조회를 요청한 경우(id 가 정수)
- id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우

id 자리에 정수가 아닌 값이 오는 경우는 해당 기능을 해야할 때 해야겠지만, 지금 당장은`403(Forbidden)` 이 잘 나오는지 확인하는 테스트를 수행합니다. 

## 존재하는 게시글 ID로 게시글 조회 요청

### 테스트 코드 작성

이전의 테스트 코드를 활용하여 간단하게 작성해줍니다.

```java
@Test
@DisplayName("존재하는 게시글 ID로 게시글 조회 요청")
void shouldReturnPostIfValidId() {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts/1", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    // 게시글 목록 uri
    URI postsList = URI.create(documentContext.read("$.postsList"));
    assertThat(postsList).hasPath("/api/v1/posts");
    
    List<Map<String, Object>> postsInfo = documentContext.read("$.post");
    assertThat(postsInfo.size()).isEqualTo(1);
    assertThat(postsInfo.get(0).get("id")).isEqualTo(1);
    assertThat(postsInfo.get(0).get("title")).isEqualTo("title1");
    assertThat(postsInfo.get(0).get("content")).isEqualTo("content1");
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get(0).get("createdAt")))).isBefore(LocalDateTime.now());
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get(0).get("updatedAt")))).isBefore(LocalDateTime.now());
    
    Map<String, Object> user = (Map<String, Object>) postsInfo.get(0).get("user");
    assertThat(user.get("id")).isEqualTo(1);
    assertThat(user.get("email")).isEqualTo("limvik@limvik.com");
}
```

테스트를 수행해보면, 기대했던대로 403(Forbidden)이 나옵니다.

```
expected: 200 OK
 but was: 403 FORBIDDEN
```

### SecurityConfig 수정

해당 경로를 접근할 수 있도록 id에 숫자가 올때만 SecurityConfig에 경로를 추가합니다.

```java
auth.requestMatchers(new RegexRequestMatcher("/api/v1/posts/\\d+", "GET")).permitAll();
```

정규표현식을 이용하여 id 자리에 숫자가 오는 경우만 매칭합니다.

### 테스트

모두에게 허용되었으므로, 없는 경로인 경우 404(Not Found)가 됩니다.

```
expected: 200 OK
 but was: 404 NOT_FOUND
```

### Controller 구현

간단하게 정상적인 게시글 조회 요청 시 무조건 200(OK)를 반환하도록 Controller 를 구현합니다.

```java
@GetMapping("/{id}")
public ResponseEntity<Void> getPost(@PathVariable long id) {
    return ResponseEntity.ok().build();
}
```

### 테스트

다시 테스트를 해보면, Response Status Code 에 대한 테스트는 넘어가고 body가 없어서 오류가 발생합니다.

```
java.lang.IllegalArgumentException: json string can not be null or empty
```

다시 Controller 의 나머지 부분을 구현하겠습니다.

### Controller 추가 구현

게시글 목록 링크를 응답에 포함하고, 추후 확장을 고려해서 PostResponse 클래스를 만들겠습니다. 이전에 게시글 목록 요청에 대한 응답용으로 만들었던 PostsList 하고 통일성이 없는데, 클래스 이름을 다시 고려해 보기는 해야겠습니다.

```java
@GetMapping("/{id}")
public ResponseEntity<PostResponse> getPost(@PathVariable long id,
                                            UriComponentsBuilder ucb) {
    PostResponse postResponse = postService.getPostResponse(id, ucb);
    
    if (postResponse != null) {
        return ResponseEntity.ok(postResponse);
    } else {
        return ResponseEntity.notFound().build();
    }
}
```

다음으로 필요한 것을 만듭니다.

### PostResponse

record 를 이용하여 간단하게 구현할 수 있습니다.

```java
package com.limvik.wantedpreonboardingbackend.form;

import com.limvik.wantedpreonboardingbackend.domain.Post;

import java.net.URI;

public record PostResponse(URI postsList, Post post) { }
```

### Service 구현

게시물을 id로 찾은 후 PostResponse 에 추가하여 반환합니다.

```java
public PostResponse getPostResponse(long id, UriComponentsBuilder ucb) {
    try {
        Post post = postRepository.findById(id).orElseThrow();
        return new PostResponse(buildPostsListUri(ucb), post);
    } catch (NoSuchElementException e) {
        return null;
    }
}

private URI buildPostsListUri(UriComponentsBuilder ucb) {
    return ucb
            .replacePath("/api/v1/posts")
            .build()
            .toUri();
}
```

그리고 테스트를 해보면, Post 클래스는 Map 으로 변환해야 하는데, 이전 코드 그대로 복붙해서 쓰다가 List로 변환해서 오류가 납니다. 바로 테스트를 수정하겠습니다.

### 테스트 수정

List가 아닌 Map으로 변환하도록 수정하고, password가 있으면 안되므로 포함 여부를 확인합니다.

```java
@Test
@DisplayName("존재하는 게시글 ID로 게시글 조회 요청")
void shouldReturnPostIfValidId() {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts/1", String.class);

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

테스트를 수행해보면 password를 별도로 없애주지는 않았으므로, 아래와 같이 테스트에 실패합니다.

```
expected: null
 but was: "password"
```

![디버거를 통해 본 password 프로퍼티](/assets/img/2023-08-13-making-rest-api-with-spring-boot-5-read-the-post/01-user-include-password-in-post.png)

password 가 나타나지 않도록 수정을 해야하는데, 그냥 값만 마스킹을 해서 지울지 아예 JSON 프로퍼티에서 제거할지 고민이됩니다.

### User 클래스 수정하기

생각해보면 password 를 평문으로 그대로 사용자에게 반환 할 일이 없습니다. 그런데 @JsonIgnore 를 하면 Serialization, Deserialization 모두 무시가 되어버립니다.

회원가입 할 때는 password를 받아야 하므로 Serialization 할 때만 무시할 방법을 찾아보니, @JsonInclude annotation이 있습니다. 해당 필드가 null 이거나 길이가 0인 경우 무시하도록 합니다.

```java
@JsonInclude(JsonInclude.Include.NON_EMPTY)
@Column(nullable = false)
@Length(min = 8, message = "{user.password.length}")
private String password;
```

이에 맞추어 Service 에서는 Post 인스턴스에 포함된 User 인스턴스의 password를 null 로 설정해줍니다.

```java
public PostResponse getPostResponse(long id, UriComponentsBuilder ucb) {
    try {
        Post post = postRepository.findById(id).orElseThrow();
        post.getUser().setPassword(null);
        return new PostResponse(buildPostsListUri(ucb), post);
    } catch (NoSuchElementException e) {
        return null;
    }
}
```

### 테스트

전체에 영향을 미칠만한 테스트이니 전체 테스트를 수행해보면, 새로 만든 테스트를 포함하여 모두 PASSED 가 시현되는 것을 볼 수 있습니다. 길어서 이번 테스트 결과만 추가합니다.

```
PostRequestTests > 존재하는 게시글 ID로 게시글 조회 요청 PASSED
```

완료되었으니 다음으로 넘어갑니다.

## 존재하지 않는 게시글 조회를 요청한 경우(id 가 정수)

이미 같이 구현을 해버려서 테스트를 늦게 구현하게 됩니다.

### 테스트 코드 작성

게시글이 존재하지 않을 때 게시글 목록을 요청하는 경우와 로직이 동일합니다.

```java
@Test
@DisplayName("존재하지 않는 게시글 조회를 요청한 경우(요청 ID 가 정수)")
void shouldReturnNotFoundWithNoPost() {

    ResponseEntity<String> createResponse = restTemplate
            .getForEntity("/api/v1/posts/1", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    assertThat(createResponse.getBody()).isNullOrEmpty();
}
```

### 테스트

테스트는 PASSED, 구현할게 없으므로 다음으로 넘어갑니다.

```
PostRequestTests > 존재하지 않는 게시글 조회를 요청한 경우(요청 ID 가 정수) PASSED
```

## id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우

지정한 경로가 없으므로, 처음에 계획한대로 403(Forbidden) 을 반환받는 테스트를 작성합니다.

### 테스트 코드 작성

```java
@Test
@DisplayName("id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우")
void shouldReturnForbiddenWithNoPost() {

    ResponseEntity<String> createResponse = restTemplate
            .getForEntity("/api/v1/posts/abcd", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    assertThat(createResponse.getBody()).isNullOrEmpty();

}
```

### 테스트

아 Body 에 관련 정보를 담아서 함께 주는 것은 생각을 못했습니다.

```
PostRequestTests > id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우 FAILED
    java.lang.AssertionError: 
    Expecting null or empty but was: "{"timestamp":"2023-08-13T09:32:23.130+00:00","status":403,"error":"Forbidden","path":"/api/v1/posts/abcd"}"
```

### 테스트 수정

동일하게 body 에도 403(Forbidden) 과 관련된 정보를 포함하는지 테스트하는 것으로 수정합니다.

```java
@Test
@DisplayName("id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우")
void shouldReturnForbiddenWithNoPost() {

    ResponseEntity<String> createResponse = restTemplate
            .getForEntity("/api/v1/posts/abcd", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    Number status = documentContext.read("$.status");
    assertThat(status).isEqualTo(403);
    String error = documentContext.read("$.error");
    assertThat(error).isEqualTo("Forbidden");

}
```

### 다시 테스트

다시 테스트를 해보면 PASSED 가 됩니다.

```
PostRequestTests > id 자리에 정수가 아닌 값(문자열 등)을 입력한 경우 PASSED
```

## Outro

뭔가 놓친거 같은 이 찝찝한 기분이 이어집니다. 일단 다음으로 넘어가서 하다보면 갑자기 생각날 수도 있겠죠.
