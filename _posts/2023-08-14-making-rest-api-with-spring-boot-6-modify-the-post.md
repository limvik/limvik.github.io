---
layout: post
title: Spring Boot로 REST API 만들어보기 (6) 특정 게시물 수정
categories:
- API
- REST
tags:
- Spring
- Spring Security
- JPA
- JUnit5
date: 2023-08-14 21:49 +0900
---
## Intro

이제 수정과 삭제, 2개의 과제가 남았습니다. 하지만, 문서화하고 가산점용 과제들 하는데 시간이 꽤나 걸릴 것 같습니다.

그럼 게시물 조회([링크](/posts/making-rest-api-with-spring-boot-5-read-the-post/))에 이이서 특정 게시물 수정 과제를 진행해보겠습니다.

## 요구사항

### 과제 6. 특정 게시글을 수정하는 엔드포인트

-   게시글의 ID와 수정 내용을 받아 해당 게시글을 수정하는 엔드포인트를 구현해 주세요.
-   게시글을 수정할 수 있는 사용자는 게시글 작성자만이어야 합니다.

이번에는 게시글 작성자가 게시글을 수정하는 요구사항입니다. 게시글 ID와 수정 내용(제목, 게시글 내용)을 받아 수정합니다.

## 특정 게시글 수정 엔드포인트

이번에는 특정 게시글 조회와 경로는 같지만 HTTP method만 변경하면 되겠습니다.

### 경로 및 HTTP method

`PUT` 과 `PATCH` 둘 중에 하나를 선택해야 합니다. 게시물ID, 작성자ID, 작성일시 등이 바뀔 일은 없으니 부분 변경을 의미하는 `PATCH`를 사용합니다.

> PATCH /posts/{id}

### HTTP 요청 성공시 response status code

PATCH 메서드 요청 성공 시에는 `200(OK)`로 합니다. 그리고 요청한 수정사항을 반영한 게시글 내용을 body에 포함하여 반환합니다. 

RFC 문서 상의 예제([링크](https://datatracker.ietf.org/doc/html/rfc5789#section-2.1))를 보면, 204(No Content)응답을 보냅니다. 아무래도 파일 수정이라 그런 것 같습니다.

```
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

HTTP/1.1 204 No Content
Content-Location: /file.txt
ETag: "e0023aa4f"
```

그리고 애매모호 하지만 다른 success code 도 사용할 수 있음을 언급합니다.

> The 204 response code is used because the response does not carry a message body (which a response with the 200 code would have). Note that other success codes could be used as well.

그래서 클라이언트 입장에서 생각해볼 때 정상적으로 수정되었음을 확인하고, 반영할 수 있게 `200(OK)` 응답과 함께 body 에 수정된 게시글의 내용을 포함합니다.

> PATCH 메서드가 Servlet 6.1(Tomcat 11) 버전([링크](https://tomcat.apache.org/tomcat-11.0-doc/servletapi/jakarta/servlet/http/HttpServlet.html))에서나 지원될 예정이라는 사실에 놀랐던 기억이 납니다. Spring Framework 는 3.2([링크](https://docs.spring.io/spring-framework/docs/3.2.0.RC1/reference/html/new-in-3.2.html))부터 지원됐습니다. Tomcat 10 버전 문서([링크](https://tomcat.apache.org/tomcat-10.1-doc/servletapi/jakarta/servlet/http/HttpServlet.html))에는 PATCH 메서드를 처리하는 메서드가 없는 것을 볼 수 있습니다.

### HTTP 요청 실패시 response status code

RFC 문서의 Error Handling 섹션([링크](https://datatracker.ietf.org/doc/html/rfc5789#section-2.2))을 참고해서 정의해 보겠습니다.

- 제목과 내용 모두 null 인 경우 `400(Bad Request)`, 둘 중에 하나가 있는 경우는 있는 것만 수정
- 없는 게시글 ID로 수정 요청한 경우 `404(Not Found)`
- 작성자가 아닌 사용자가 수정 요청한 경우 `401(Unauthorized)`

근데 생각해보니 여태 왜 status code를 결정하면서 RFC 문서를 안봤는지 모르겠습니다. RFC 문서를 보니 PATCH 메서드를 제대로 구현하려면 한참 걸리겠습니다.

### 요청 양식

요청 시에는 수정 대상과 수정 내용만 포함하면 됩니다.

> 게시물 ID(필수), 제목/내용(둘 중 하나 필수)

### 응답 양식

응답 시에는 게시물과 관련된 모든 정보를 반환합니다.

>게시물 ID, 제목, 내용, 작성 시간, 수정 시간, 작성자

## 테스트 및 구현 목록

이번에도 일단 목록을 나열해 봅니다.

- 유효한 토큰으로 자신이 작성한 게시글 수정 요청
	- 제목과 내용이 모두 수정 - 200(OK)
	- 제목만 수정 - 200(OK)
	- 내용만 수정 - 200(OK)
	- 미수정 - 400(Bad Request)
- 유효한 토큰으로 존재하지 않는 게시글 수정 요청 - 404(Not Found)
- 유효한 토큰으로 다른 사람의 게시글 수정 요청 - 401(Unauthorized)

아래는 Spring Security 에 의하여 잘 처리되는지 테스트만 작성합니다.

- 유효한 토큰으로 게시글 ID 위치에 문자열을 입력한 후 게시글 수정 요청 - 403(Forbidden)
- 토큰 없이 수정 요청 - 403(Forbidden)

그리고 게시글 ID 등 사용자가 임의로 수정할 수 없는 부분을 request body 에 포함하여 수정 요청할 경우 무시하고 제목과 내용만 수정합니다.

이번에는 좀 가지수가 늘었습니다.

## 유효한 토큰으로 자신이 작성한 게시글의 수정 요청

### 테스트 코드 작성

이번에는 Parameterized Test로 수행하는게 좋겠습니다.

csv source 로 자리를 비워두면 null 이 되는지 확인하기 위해 테스트 로직 없이 실행해 봅니다.

```java
@ParameterizedTest
@DisplayName("유효한 토큰으로 자신이 작성한 게시글의 수정 요청")
@CsvSource({"수정된 제목1, 수정된 내용1", "수정된 제목2,", ", 수정된 내용3", ","})
void shouldModifyPostAndReturnModifiedPostIfValidTokenAndRequestedByPostWriter
        (String modifiedTitle, String modifiedContent) {

}
```

원하는 대로 null 이 입력되는 것을 볼 수 있습니다.

```
[1] modifiedTitle=수정된 제목1, modifiedContent=수정된 내용1
[2] modifiedTitle=수정된 제목2, modifiedContent=null
[3] modifiedTitle=null, modifiedContent=수정된 내용3
[4] modifiedTitle=null, modifiedContent=null
```

계속해서 테스트 로직도 작성합니다.

```java
@ParameterizedTest
@DisplayName("유효한 토큰으로 자신이 작성한 게시글의 수정 요청")
@CsvSource({"수정된 제목1, 수정된 내용1", "수정된 제목2,", ", 수정된 내용3", ","})
void shouldModifyPostAndReturnModifiedPostIfValidTokenAndRequestedByPostWriter
        (String modifiedTitle, String modifiedContent) {
    postRepository.saveAll(getNewPosts(1));

    Post RequestedPost = Post.builder()
            .title(modifiedTitle)
            .content(modifiedContent)
            .build();

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Post> request = new HttpEntity<>(RequestedPost, headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/1", HttpMethod.PATCH, request, String.class);
    if (modifiedTitle == null && modifiedContent == null) {
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
    } else {
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

        Post savedModifiedPost = postRepository.findById(1L).get();
        assertThat(savedModifiedPost.getId()).isEqualTo(1);
        assertThat(savedModifiedPost.getTitle()).isEqualTo(Objects.requireNonNullElse(modifiedTitle, "title1"));
        assertThat(savedModifiedPost.getContent()).isEqualTo(Objects.requireNonNullElse(modifiedContent, "content1"));
        assertThat(savedModifiedPost.getCreatedAt()).isBefore(savedModifiedPost.getUpdatedAt());
        assertThat(savedModifiedPost.getUser().getId()).isEqualTo(1);
    }
}
```

### 테스트

테스트를 해보면 다음과 같이 TestRestTemplate 에서 PATCH 메서드를 지원하지 않아 오류가 발생합니다.

```java
org.springframework.web.client.ResourceAccessException: I/O error on PATCH request for "http://localhost:13339/api/v1/posts/1": Invalid HTTP method: PATCH

Invalid HTTP method: PATCH
java.net.ProtocolException: Invalid HTTP method: PATCH
```

### TestRestTemplate 에서 PATCH 메서드를 사용하기 위한 설정

이번에는 IntelliJ 의 AI Assistant 의 도움을 받아봅니다.

Apache HttpClient([maven 저장소 링크](https://mvnrepository.com/artifact/org.apache.httpcomponents.client5/httpclient5/5.2.1)) 라이브러리를 추가합니다.

어차피 테스트에서만 사용할 것이기 때문에 testImplementation 으로 선언합니다.

```groovy
testImplementation 'org.apache.httpcomponents.client5:httpclient5:5.2.1'
```

그리고 TestConfig 클래스를 추가하여 TestRestTemplate 반환 시 설정을 추가합니다.

```java
package com.limvik.wantedpreonboardingbackend;

import org.apache.hc.client5.http.classic.HttpClient;
import org.apache.hc.client5.http.impl.classic.HttpClientBuilder;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;

@TestConfiguration
public class TestConfig {

    @Bean
    public TestRestTemplate testRestTemplate(RestTemplateBuilder builder) {

        HttpClient httpClient = HttpClientBuilder.create().build();
        TestRestTemplate testRestTemplate = new TestRestTemplate(builder.requestFactory(()
                -> new HttpComponentsClientHttpRequestFactory(httpClient)));

        return testRestTemplate;
    }
}
```

### 테스트

다시 테스트를 수행하기 전에 @Disabled 처리를 하고, 나머지 테스트에 영향이 없는지 전체 테스트를 수행해본 결과 이상이 없습니다.

그리고 다시 @Disabled를 제거하고 테스트 해보면 아래와 같이 Spring Security 로 인해서 403(Forbidden)이 출력됩니다.

```
expected: 200 OK
 but was: 403 FORBIDDEN

expected: 400 BAD_REQUEST
 but was: 403 FORBIDDEN
```

### SecurityConfig 수정

게시물 수정에 대해 PATCH 메서드로 접근 시 필요한 권한을 지정합니다.

```java
auth.requestMatchers(new RegexRequestMatcher("/api/v1/posts/\\d+", "PATCH")).hasRole("USER");
```

위 코드를 다음과 같이 SecurityFilterChain 에 추가합니다.

```java
@Bean
public SecurityFilterChain postFilterChain(HttpSecurity http) throws Exception {
    return http
            .securityMatcher("/api/v1/posts/**")
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/posts", "POST")).hasRole("USER");
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/posts", "GET")).permitAll();
                auth.requestMatchers(new RegexRequestMatcher("/api/v1/posts/\\d+", "PATCH")).hasRole("USER");
                auth.requestMatchers(new RegexRequestMatcher("/api/v1/posts/\\d+", "GET")).permitAll();})
            .csrf(AbstractHttpConfigurer::disable)
            .logout(AbstractHttpConfigurer::disable)
            .anonymous(AbstractHttpConfigurer::disable)
            .sessionManagement(config -> config.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterAfter(new XAuthAuthenticationFilter(getJwtProviderManager()),
                    RequestCacheAwareFilter.class)
            .build();
}
```

### 테스트

다시 테스트를 해보면, 이번에는 이미 경로는 있는데 HTTP 메서드가 일치하는 것이 없어 405(Method Not Allowed)가 반환됩니다.

```
expected: 200 OK
 but was: 405 METHOD_NOT_ALLOWED

expected: 400 BAD_REQUEST
 but was: 405 METHOD_NOT_ALLOWED
```

### 무조건 200(OK)를 반환하는 Controller 구현

간단하게 테스트를 위해 PATCH 메서드 경로를 구현합니다.

```java
@PatchMapping("/{id}")
public ResponseEntity<Post> modifyPost(@PathVariable long id) {
    return ResponseEntity.ok().build();
}
```

### 테스트

그리고 테스트를 하면, 아래와 같은 결과가 나옵니다.

```
expected: "수정된 제목1"
 but was: "title1"

expected: "수정된 제목2"
 but was: "title1"

expected: "수정된 내용3"
 but was: "content1"

expected: 400 BAD_REQUEST
 but was: 200 OK
```

### Controller 추가 구현

이제 Controller 부터 게시글의 수정된 사항이 반영될 수 있도록 구현하겠습니다.

```java
@PatchMapping("/{id}")
public ResponseEntity<Post> modifyPost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId,
                                       @RequestBody Post modifyRequestedPost) {

    if (modifyRequestedPost.getTitle() == null && modifyRequestedPost.getContent() == null) {
        return ResponseEntity.badRequest().build();
    }

    modifyRequestedPost.setId(postId);
    modifyRequestedPost.setUser(user);
    
    Post savedModifiedPost = postService.updatePost(modifyRequestedPost);
    return ResponseEntity.ok(savedModifiedPost);

}
```

### Service 구현

JPA 지식이 짧아서 @NotBlank 가 지정된 title과 content 이 둘 중 하나만 null 일 때를 깔끔하게 처리할 방법이 잘 떠오르지 않습니다. 제출이 급하니 제출 후에 더 좋은 방법을 찾아봐야겠습니다.

```java
@Transactional
public Post updatePost(Post modifyRequestedPost) {
    Post existedPost = postRepository.findById(modifyRequestedPost.getId()).orElseThrow();

    return saveModifiedPost(existedPost, modifyRequestedPost);
}

private Post saveModifiedPost(Post existedPost, Post modifyRequestedPost) {

    String modifyRequestedTitle = modifyRequestedPost.getTitle();
    String modifyRequestedContent = modifyRequestedPost.getContent();

    if (existedPost.getUser().getId().equals(modifyRequestedPost.getUser().getId())) {
        if (modifyRequestedTitle != null)
            existedPost.setTitle(modifyRequestedTitle);
        if (modifyRequestedContent != null)
            existedPost.setContent(modifyRequestedContent);
        return existedPost;
    }

    return null;
}
```

코드가 썩 마음에 들지는 않지만 테스트를 진행해봅니다.

### 테스트

테스트를 진행하면, PASSED가 시현되는 것을 볼 수 있습니다.

```
[1] modifiedTitle=수정된 제목1, modifiedContent=수정된 내용1 PASSED
[2] modifiedTitle=수정된 제목2, modifiedContent=null PASSED
[3] modifiedTitle=null, modifiedContent=수정된 내용3 PASSED
[4] modifiedTitle=null, modifiedContent=null PASSED
```

## 유효한 토큰으로 존재하지 않는 게시글 수정 요청

실패 뜨는 것을 보려고 앞에서는 테스트 작성한 것만 통과하는 코드를 작성했었습니다. 다음 테스트 코드를 작성하고 확인해 보겠습니다.

### 테스트 코드 작성

```java
@Test
@DisplayName("유효한 토큰으로 존재하지 않는 게시글 수정 요청")
void shouldReturnNotFoundIfRequestModifyWithNoneExistPostId() {
    postRepository.saveAll(getNewPosts(1));

    Post RequestedPost = Post.builder()
            .title("수정된 제목")
            .content("수정된 내용")
            .build();

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Post> request = new HttpEntity<>(RequestedPost, headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/99", HttpMethod.PATCH, request, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

TestRestTemplate 의 exchage 메서드에서 id를 99로 설정한 것 말고는 특이한 것은 없습니다.

### 테스트

Service에서 Repository의 findById() 메서드로 Post 를 불러올 때 Optional 을 사용하여 NoSuchElementException 이 던져집니다.

```
java.util.NoSuchElementException: No value present
```

### Service 와 Controller 수정하기

Service 에서 NoSuchElementException이 던져졌을 때 404(Not Found)를 반환하도록 수정합니다.

먼저 Service 에서 찾는 게시글이 없을 경우 null 을 반환합니다.

```java
@Transactional
public Post updatePost(Post modifyRequestedPost) {
    try {
        Post existedPost = postRepository.findById(modifyRequestedPost.getId()).orElseThrow();
        return saveModifiedPost(existedPost, modifyRequestedPost);
    } catch (NoSuchElementException e) {
        return null;
    }
}
```

그리고 Controller 에서는 null 이 반환될 경우 404(Not Found)를 반환하도록 수정합니다.

```java
@PatchMapping("/{id}")
public ResponseEntity<Post> modifyPost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId,
                                       @RequestBody Post modifyRequestedPost) {

    if (modifyRequestedPost.getTitle() == null && modifyRequestedPost.getContent() == null) {
        return ResponseEntity.badRequest().build();
    }

    modifyRequestedPost.setId(postId);
    modifyRequestedPost.setUser(user);

    Post savedModifiedPost = postService.updatePost(modifyRequestedPost);
    if (savedModifiedPost == null)
        return ResponseEntity.notFound().build();

    return ResponseEntity.ok(savedModifiedPost);

}
```

### 테스트

다시 테스트를 해보면 PASSED 되는 것을 볼 수 있습니다.

```
유효한 토큰으로 존재하지 않는 게시글 수정 요청 PASSED
```

## 유효한 토큰으로 다른 사람의 게시글 수정 요청

### 테스트 코드 작성

ID가 1인 사용자가 작성한 ID가 1인 게시물을 ID가 2인 사용자의 토큰으로 수정을 요청하면 401(Unauthorized)를 반환하는지 검사하도록 테스트 코드를 작성합니다.

```java
@Test
@DisplayName("유효한 토큰으로 다른 사람의 게시글 수정 요청")
void shouldReturnUnauthorizedIfRequestModifyTheNotOwnPost() {
    postRepository.saveAll(getNewPosts(1));

    Post RequestedPost = Post.builder()
            .title("수정된 제목")
            .content("수정된 내용")
            .build();

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(2L).email("limvik2@limvik2.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Post> request = new HttpEntity<>(RequestedPost, headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/1", HttpMethod.PATCH, request, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
}
```

### 테스트

테스트를 해보면, 404(Not Found)가 나옵니다.

```
expected: 401 UNAUTHORIZED
 but was: 404 NOT_FOUND
```

Service 에서 수정 요청한 게시물의 작성자 ID와 수정 요청자의 토큰에 있던 ID를 비교하여 다르면 null 을 반환하여 404를 반환하는 로직이 작성되어 있기 때문입니다.

### Service 수정

Exception을 던져서 각 상황을 나타내기로 합니다.

```java
@Transactional
public Post updatePost(Post modifyRequestedPost) throws AccessDeniedException, NoSuchElementException {
    Post existedPost = postRepository.findById(modifyRequestedPost.getId()).orElseThrow();
    return saveModifiedPost(existedPost, modifyRequestedPost);
}

private Post saveModifiedPost(Post existedPost, Post modifyRequestedPost) {

    String modifyRequestedTitle = modifyRequestedPost.getTitle();
    String modifyRequestedContent = modifyRequestedPost.getContent();

    if (existedPost.getUser().getId().equals(modifyRequestedPost.getUser().getId())) {
        if (modifyRequestedTitle != null)
            existedPost.setTitle(modifyRequestedTitle);
        if (modifyRequestedContent != null)
            existedPost.setContent(modifyRequestedContent);
        return existedPost;
    }
    throw new AccessDeniedException("게시글 작성자만 수정할 수 있습니다.");
}
```

### Controller 수정하기

Exception 의 종류에 따라 다른 응답을 하도록 수정합니다.

```java
@PatchMapping("/{id}")
public ResponseEntity<Object> modifyPost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId,
                                       @RequestBody Post modifyRequestedPost) {

    if (modifyRequestedPost.getTitle() == null && modifyRequestedPost.getContent() == null) {
        // 수정 사항 없는 수정 요청
        return ResponseEntity.badRequest().build();
    }

    modifyRequestedPost.setId(postId);
    modifyRequestedPost.setUser(user);

    Post savedModifiedPost;

    try {
        savedModifiedPost = postService.updatePost(modifyRequestedPost);
    } catch (AccessDeniedException e) {
        // 다른 사람의 게시물에 대한 수정 요청
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(e.getMessage());
    } catch (NoSuchElementException e) {
        // 없는 게시물에 대한 수정 요청
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getLocalizedMessage());
    }

    // 자신의 게시물에 대한 정상적인 수정 요청
    return ResponseEntity.ok(savedModifiedPost);

}
```

### 테스트 수정 및 실행

큰 수정 사항 없이 응답코드가 401(Unauthorized) 인지 확인한 후에 body에 에러 메시지가 포함되어 있는지 확인합니다.

```java
assertThat(createResponse.getBody().toString()).contains("게시글 작성자만 수정할 수 있습니다.");
```

그러면 이상 없이 PASSED 되는 것을 볼 수 있습니다.

```
PostRequestTests > 유효한 토큰으로 다른 사람의 게시글 수정 요청 PASSED
```

다음으로 넘어가겠습니다. Spring Security 가 알아서 응답해서 할 일이 없는 경우들 입니다.

## 유효한 토큰으로 게시글 ID 위치에 문자열을 입력한 후 게시글 수정 요청

### 테스트 코드 작성

```java
@Test
@DisplayName("유효한 토큰으로 게시글 ID 위치에 문자열을 입력한 후 게시글 수정 요청")
void shouldReturnForbiddenIfNotPostIdFormat() {
    postRepository.saveAll(getNewPosts(1));

    Post RequestedPost = Post.builder()
            .title("수정된 제목")
            .content("수정된 내용")
            .build();

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Post> request = new HttpEntity<>(RequestedPost, headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/abcd", HttpMethod.PATCH, request, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    forbiddenBodyMessageTest(createResponse.getBody());
}

private void forbiddenBodyMessageTest(String body) {  
    DocumentContext documentContext = JsonPath.parse(body);  
  
    Number status = documentContext.read("$.status");  
    assertThat(status).isEqualTo(403);  
    String error = documentContext.read("$.error");  
    assertThat(error).isEqualTo("Forbidden");  
}
```

403(Forbidden) 메시지 검사하는 중복 코드는 private 메서드로 뺐습니다.

### 테스트

```
유효한 토큰으로 게시글 ID 위치에 문자열을 입력한 후 게시글 수정 요청 PASSED
```

흠 자꾸 뻘짓 하는 느낌이... Spring Security 잘못 변경했을 때 라던가... 를 대비할 수 있지 않을까? 라고 합리화 해봅니다.

## 토큰 없이 게시글 수정 요청

### 테스트 코드 작성

토큰 없이 게시글 수정 요청을 하면 Spring Security 에서 401(Unauthorized)를 반환하는지 테스트를 작성합니다.

```java
@Test
@DisplayName("토큰 없이 게시글 수정 요청")
void shouldNotModifyPostAndReturnUnauthorizedIfNoToken() {
    postRepository.saveAll(getNewPosts(1));

    Post RequestedPost = Post.builder()
            .title("수정된 제목")
            .content("수정된 내용")
            .build();

    HttpEntity<Post> request = new HttpEntity<>(RequestedPost);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/1", HttpMethod.PATCH, request, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    forbiddenBodyMessageTest(createResponse.getBody());

}
```

### 테스트

마찬가지로 이상없이 PASSED가 시현됩니다.

```
토큰 없이 게시글 수정 요청 PASSED
```

## 기능 테스트 및 수정

Postman 을 이용하여 첫 번째 테스트로 정상적인 수정 요청을 하자마자 반환은 잘 되는데 password가 반환되는 것을 보고 깜짝 놀라서 관련 코드를 수정합니다.

### Controller 수정

Service에서 null을 해버리면 데이터베이스에 반영이 되기 때문에, Controller 에서 반환하기 전에 password를 null로 만듭니다. 하는김에 별로 필요 없는 사용자 가입일시도 제거합니다.

```java
savedModifiedPost.getUser().setPassword(null);
savedModifiedPost.getUser().setCreatedAt(null);
// 자신의 게시물에 대한 정상적인 수정 요청
return ResponseEntity.ok(savedModifiedPost);
```

### User 수정

사용자 가입일시도 null 이면 JSON이 되지 않게 하기 위해 필드에 annotation을 추가합니다.

```java
@CreationTimestamp
@JsonInclude(JsonInclude.Include.NON_EMPTY)
private LocalDateTime createdAt;
```

### 테스트 수정

assertThrows 메서드를 이용하여, 사용자 비밀번호와 가입일시가 JSON 프로퍼티로 포함이 되지 않은 것을 확인합니다.

```java
assertThrows(LazyInitializationException.class, () -> savedModifiedPost.getUser().getPassword());
assertThrows(LazyInitializationException.class, () -> savedModifiedPost.getUser().getCreatedAt());
```

그리고 테스트 결과 PASSED가 시현됩니다. 결과는 굳이 포함시키지 않겠습니다.

### 다시 기능 테스트

Postman으로 다시 수정을 요청해봅니다.

![Postman 으로 게시물 수정 요청 결과 확인](/assets/img/2023-08-14-making-rest-api-with-spring-boot-6-modify-the-post/01-fix-return-with-password.png)

이상 없이 사용자의 비밀번호와 가입일시가 포함되지 않는 것을 볼 수 있습니다.

나머지는 PR 작성하면서 테스트 해보겠습니다.

## Outro

드디어 1개가 남았습니다. 다행히도 내일이 광복절이네요.

아직도 수정하고 싶은 예외적 상황들이 보이기는 하는데, 현실적으로 마지막 기능을 마무리하고 문서화 하는데 시간을 써야 제출이라도 해볼 수 있을 것 같습니다.
