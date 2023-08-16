---
layout: post
title: Spring Boot로 REST API 만들어보기 (7) 특정 게시물 삭제
categories:
- API
- REST
tags:
- Spring
- Spring Security
- JPA
- JUnit5
- REST
date: 2023-08-15 12:26 +0900
---
## Intro

게시물 수정([링크](/posts/making-rest-api-with-spring-boot-6-modify-the-post/))을 마쳤고, 다음은 삭제입니다. 드디어 기능은 마지막입니다.

## 요구사항

### 과제 7. 특정 게시글을 삭제하는 엔드포인트

- 게시글의 ID를 받아 해당 게시글을 삭제하는 엔드포인트를 구현해 주세요.
- 게시글을 삭제할 수 있는 사용자는 게시글 작성자만이어야 합니다.

게시글 수정과 거의 비슷하지만, 게시물 제목과 내용의 수정에 따른 동작은 고려할 필요가 없으니 작업량은 더 줄어들겠습니다.

## 특정 게시글 삭제 엔드포인트

수정과 마찬가지로 게시물에 대한 HTTP method 만 변경하면 됩니다.

### 경로 및 HTTP method

삭제는 간단하게 `DELETE` 를 선택할 수 있습니다.

> DELETE /posts/{id}

### HTTP 요청 성공시 response status code

기존 리소스의 존재 자체가 사라져버리니 `204(No Content)`를 반환합니다.

참고로 RFC 문서([링크](https://www.rfc-editor.org/rfc/rfc9110.html#name-delete))를 보면 돌려줄 내용이 있으면 200(OK)를 사용하라고 합니다.

>If a DELETE method is successfully applied, the origin server SHOULD send
>
>- a `202 (Accepted)` status code if the action will likely succeed but has not yet been enacted,
>
>- a `204 (No Content)` status code if the action has been enacted and no further information is to be supplied, or
>
>- a `200 (OK)` status code if the action has been enacted and the response message includes a representation describing the status.


### HTTP 요청 실패시 response status code

- 다른 사람의 게시글 삭제를 요청하는 경우 `401(Unauthorized)`
- 존재하지 않는 게시글 삭제를 요청하는 경우 `404(Not Found)`
- path 상의 게시글 ID 위치에 문자열을 포함한 경우 `403(Forbidden)`
- 토큰 없이 게시글 삭제 요청한 경우 `403(Forbidden)`

## 유효한 토큰으로 자신이 작성한 게시글 삭제 요청

### 테스트 코드 작성

JWT를 헤더에 포함하여 DELETE 메서드를 이용한 HTTP 요청을 보내고, 204(No Content) 응답과 body는 비어있는지 확인하는 테스트를 작성합니다.

```java
@Test  
@DisplayName("유효한 토큰으로 자신이 작성한 게시글 삭제 요청")  
void shouldDeletePostByPostWriter() {  
    postRepository.saveAll(getNewPosts(1));  
  
    HttpHeaders headers = new HttpHeaders();  
    String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());  
    headers.set("X-AUTH-TOKEN", jwt);  
    HttpEntity<Void> request = new HttpEntity<>(headers);  
  
    ResponseEntity<Void> createResponse = restTemplate  
  .exchange("/api/v1/posts/1", HttpMethod.DELETE, request, Void.class);  
  
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);  
    assertThat(createResponse.getBody()).isNull();  
}
```

### 테스트

테스트 결과는 403(Forbidden) 입니다.

```
expected: 204 NO_CONTENT
 but was: 403 FORBIDDEN
```

### SecurityConfig 수정

지정한 경로에 대해 DELETE 메서드를 사용자에게 허용하도록 수정합니다.

아래 코드를 기존의 SecurityFilterChain에 추가합니다.

```java
auth.requestMatchers(new RegexRequestMatcher("/api/v1/posts/\\d+", "DELETE")).hasRole("USER");
```

### 테스트

그러면 Controller 에서 DELETE Method 에 대해 mapping 된 것이 없어 405(Method Not Allowed)가 반환됩니다.

```
expected: 204 NO_CONTENT
 but was: 405 METHOD_NOT_ALLOWED
```

### Controller 수정

DELETE 메서드를 mapping 하고, 204(No Content)를 반환하도록 코드를 작성합니다.

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deletePost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId) {
    return ResponseEntity.noContent().build();
}
```

### 테스트

간단하게 한 번 확인하고 갑니다.

```
유효한 토큰으로 자신이 작성한 게시글 삭제 요청 PASSED
```

### Controller 추가 수정

Service에 요청할 작업을 Controller 에서 추가적으로 작성합니다.

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deletePost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId) {
    
    postService.deletePost(user, postId);
    
    return ResponseEntity.noContent().build();
}
```

현재는 단순하게 삭제만 하도록 합니다.

### Service 수정

마찬가지로 단순하게 삭제를 합니다.

```java
@Transactional
public void deletePost(User user, long postId) {
    postRepository.deleteById(postId);
}
```

### 테스트 보완 및 실행

실제로 데이터베이스에서 지워졌는지 확인하는 테스트를 수행하도록 보완합니다.

```java
assertThrows(NoSuchElementException.class, () -> postRepository.findById(1L).orElseThrow());
```

그리고 테스트를 실행해보면 이상 없이 PASSED가 시현되는 것을 볼 수 있습니다.

```java
유효한 토큰으로 자신이 작성한 게시글 삭제 요청 PASSED
```

## 유효한 토큰으로 다른 사람의 게시글 삭제 요청

### 테스트 코드 작성

이번에는 다른 사람의 게시글에 대해 삭제를 요청하면 401(Unauthorized)를 반환하는 테스트를 작성합니다.

```java
@Test
@DisplayName("유효한 토큰으로 다른 사람의 게시글 삭제 요청")
void shouldNotDeletePostByNotPostWriter() {
    postRepository.saveAll(getNewPosts(1));

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(2L).email("limvik2@limvik2.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Void> request = new HttpEntity<>(headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/1", HttpMethod.DELETE, request, String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
    assertThat(createResponse.getBody().toString()).contains("다른 사람의 게시글을 삭제할 수 없습니다.");

    assertThat(postRepository.findById(1L).isPresent()).isTrue();

}
```

### 테스트

테스트를 수행하면 코드를 작성했던 대로 204(No Content)가 반환됩니다.

```
expected: 401 UNAUTHORIZED
 but was: 204 NO_CONTENT
```

### Controller 수정

수정할 때와 통일성있게 다른 사람의 게시글을 삭제 요청하면 Service에서 AccessDeniedException 을 던지고, Controller에서 201(Unauthorized)를 반환하도록 처리합니다.

```java
@DeleteMapping("/{id}")
public ResponseEntity<String> deletePost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId) {
    try {
        postService.deletePost(user, postId);
    } catch (AccessDeniedException e) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(e.getMessage());
    }

    return ResponseEntity.noContent().build();
}
```

### Service 수정

기존의 post를 조회하여 게시글이 존재하는 경우에만 삭제되도록 수정합니다.

```java
@Transactional
public void deletePost(User user, long postId) throws AccessDeniedException {
    if (postRepository.findByIdAndUserId(postId, user.getId()).isPresent())
        postRepository.deleteById(postId);
    else {
        throw new AccessDeniedException("다른 사람의 게시글을 삭제할 수 없습니다.");
    }
}
```

### Repository 수정

findByIdAndUserId 는 기본 메서드가 아니기 때문에 추가해줍니다.

```java
package com.limvik.wantedpreonboardingbackend.repository;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface PostRepository extends JpaRepository<Post, Long> {
    Optional<Post> findByIdAndUserId(Long id, Long userId);

}

```

### 테스트

그리고 테스트를 수행해보면, PASSED를 볼 수 있습니다.

```
유효한 토큰으로 다른 사람의 게시글 삭제 요청 PASSED
```

## 유효한 토큰으로 존재하지 않는 게시글 삭제를 요청하는 경우

### 테스트 코드 작성

존재하지 않는 게시글에 대한 삭제 요청은 404(Not Found)를 반환하는지 확인하는 테스트를 작성합니다.

```java
    @Test
    @DisplayName("유효한 토큰으로 존재하지 않는 게시글 삭제를 요청하는 경우")
    void shouldNotDeletePostIfNotExistPost() {
        postRepository.saveAll(getNewPosts(1));

        HttpHeaders headers = new HttpHeaders();
        String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
        headers.set("X-AUTH-TOKEN", jwt);
        HttpEntity<Void> request = new HttpEntity<>(headers);

        ResponseEntity<String> createResponse = restTemplate
                .exchange("/api/v1/posts/2", HttpMethod.DELETE, request, String.class);

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);

    }
```

### 테스트

현재 게시글을 찾지 못하면, AccessDeniedException 이 던져지므로 401(Unauthorized)가 반환됩니다.

```
expected: 404 NOT_FOUND
 but was: 401 UNAUTHORIZED
```

### Controller 수정

게시글을 찾지 못한 경우 수정할 때와 통일하여 NoSuchElementException을 던진 것을 404(Not Found)로 반환하도록 처리합니다.

```java
@DeleteMapping("/{id}")
public ResponseEntity<String> deletePost(@AuthenticationPrincipal User user,
                                       @PathVariable(name = "id") long postId) {
    try {
        postService.deletePost(user, postId);
    } catch (AccessDeniedException e) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(e.getMessage());
    } catch (NoSuchElementException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getLocalizedMessage());
    }

    return ResponseEntity.noContent().build();
}
```

### Service 수정

JPA 메서드를 만들어서 사용해볼 기회라고 생각했는데, 아니었습니다. 너무 근시안적...

```java
@Transactional
public void deletePost(User user, long postId) throws AccessDeniedException, NoSuchElementException {
    Post post = postRepository.findById(postId).orElseThrow();
    if (Objects.equals(post.getUser().getId(), user.getId()))
        postRepository.deleteById(postId);
    else {
        throw new AccessDeniedException("다른 사람의 게시글을 삭제할 수 없습니다.");
    }
}
```

### 테스트

그리고 다시 테스트를 수행해보면, PASSED 를 볼 수 있습니다.

```
유효한 토큰으로 존재하지 않는 게시글 삭제를 요청하는 경우 PASSED
```

다음 부터는 Spring Security 에 의해 403(Forbidden) 반환이 잘 되는지 확인합니다.

## 유효한 토큰으로 게시글 ID 위치에 문자열을 사용하여 게시글 삭제 요청

### 테스트 코드 작성

```java
@Test
@DisplayName("유효한 토큰으로 게시글 ID 위치에 문자열을 사용하여 게시글 삭제 요청")
void shouldNotDeleteAndReturnForbiddenIfNotPostIdFormatWithValidToken() {
    postRepository.saveAll(getNewPosts(1));

    HttpHeaders headers = new HttpHeaders();
    String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
    headers.set("X-AUTH-TOKEN", jwt);
    HttpEntity<Post> request = new HttpEntity<>(headers);

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/abcd", HttpMethod.DELETE, request, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    forbiddenBodyMessageTest(createResponse.getBody());

}
```

### 테스트

기대했던대로 PASSED 가 보입니다.

```
유효한 토큰으로 게시글 ID 위치에 문자열을 사용하여 게시글 삭제 요청 PASSED
```

## 토큰 없이 게시글 삭제 요청

### 테스트 코드 작성

```java
@Test
@DisplayName("토큰 없이 게시글 삭제 요청")
void shouldNotDeletePostAndReturnForbiddenIfNoToken() {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse = restTemplate
            .exchange("/api/v1/posts/1", HttpMethod.PATCH, null, String.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    forbiddenBodyMessageTest(createResponse.getBody());

}
```

### 테스트

마찬가지로 이상없이 PASSED 됩니다.

```
토큰 없이 게시글 삭제 요청 PASSED
```

## 기능 테스트

글에는 포함하지 않았지만 전체 테스트를 모두 통과하는지 확인했고, Postman 으로 간단하게 삭제가 되는지 확인해보겠습니다.

Postman으로 자신의 글에 대해 삭제를 요청하면 204(No Content)를 반환하고, Body는 없습니다.

![Postman으로 삭제를 요청하여 성공한 화면](/assets/img/2023-08-15-making-rest-api-with-spring-boot-7-delete-the-post/01-delete-the-post.png)

그리고 오름차순으로 게시글 목록을 조회 해보면, 1번 글이 사라진 것을 볼 수 있습니다.

![Postman 에서 게시글 목록을 조회한 화면](/assets/img/2023-08-15-making-rest-api-with-spring-boot-7-delete-the-post/02-get-posts-list.png)

## Outro

만족스럽지는 않지만 기능 요구사항은 마무리가 됐습니다. 문서화에 시간이 얼마나 걸릴지 해본적이 없어서 가늠이 안되니 빨리 해봐야겠습니다.
