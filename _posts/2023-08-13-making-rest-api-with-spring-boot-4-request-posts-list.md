---
layout: post
title: Spring Boot로 REST API 만들어보기 (4) 게시글 목록 요청
categories:
- API
- REST
tags:
- Spring
- JPA
- JUnit5
date: 2023-08-13 13:03 +0900
---
## Intro

게시글 생성([링크](/posts/making-rest-api-with-spring-boot-3-create-a-new-post/))을 완료했고, 다음 과제로 넘어갑니다. 가산점을 받는 마지막 날이 되니 아쉬운 느낌도 듭니다.

## 요구사항

### 과제 4. 게시글 목록을 조회하는 엔드포인트

-   반드시 Pagination 기능을 구현해 주세요.

페이지(page) 번호와 한 페이지에 담길 게시물의 갯수(size)를 쿼리 파라미터(query parameter)로 받아서 반환하도록 구현하겠습니다.

## 게시글 목록 조회 엔드포인트

요구사항이 구체적이지 않아서, 이전에 다음 과제인 현재 과제를 생각할 때 당연히 로그인하지 않은 사용자도 글을 불러오면 되겠다고 생각했고 그대로 진행할 예정입니다.

### 경로 및 자원명

경로(path)는 리소스(resource) 이름이 있는 `/posts` 로 지정하고, 쿼리 파리미터를 받아 `/posts?size=숫자&page=숫자` 와 같이 사용합니다.

>/posts
>
>/posts?size=숫자&page=숫자
>
>/posts?size=숫자
>
>/posts?page=숫자

추가적으로 정렬을 위한 쿼리 파라미터도 지정 가능하도록 합니다. 

>/posts?size=숫자&page=숫자&sort=컬럼명,desc
>
>/posts?size=숫자&page=숫자&sort=컬럼명,asc

그러면, Controller에서 Spring Data 의 Pageable([Github 링크](https://github.com/spring-projects/spring-data-commons/blob/main/src/main/java/org/springframework/data/domain/Pageable.java)) 로 받을 수 있습니다. 구현체인 PageRequest([Github 링크](https://github.com/spring-projects/spring-data-commons/blob/main/src/main/java/org/springframework/data/domain/PageRequest.java))를 보면 `size`, `page` 필드는 부모인 AbstractPageRequest 에 있고, `sort` 는 PageRequest 에서 볼 수 있습니다.

여튼, 다양한 파라미터가 있지만 경로는 `/posts` 입니다.

다음으로 HTTP 와 관련한 내용들을 정의하겠습니다.

### HTTP method

데이터를 가져오는 일이니 이번에도 큰 어려움 없이 `GET` 을 선택합니다.

### HTTP 요청 성공 시 response status code

`GET` 메서드 성공 시에는 `200(OK)`를 반환하고, 게시글을 반환합니다.

하지만 콘텐츠가 없는 경우도 있으니, 콘텐츠가 없다면 `204(No Content)`를 반환합니다.

### HTTP 요청 실패 시 response status code

page에 숫자가 아닌 문자를 넣는 경우, `400(Bad Request)` 를 반환하는 것으로 합니다. 이외에는 숫자이기만 하면 기본값으로 정상 반환합니다.

### 응답 양식

GET는 body가 없으므로, 응답 양식만 정하면 됩니다.

게시판에 대한 정보를 주어야 합니다. 요구사항이 너무나도 열려있는데 질문은 안받아서 뭘 고를까 고민이됩니다.

제가 게시판 만드는 걸 상상하면서, 프론트 작업할 때 편하겠다 싶은거를 모두 추가합니다.

> 총 페이지 수, 총 게시글 수, 현재 페이지 번호, 첫 페이지 링크, 마지막 페이지 링크, 이전 페이지 링크, 다음 페이지 링크, [게시글 id, 게시글 제목, 게시글 작성자, 게시글 작성일, 게시글 링크] 

대괄호([ ])로 표시해둔 개별 게시글에 대한 정보는 여러개가 올 수 있습니다. 그리고 REST API 의 핵심 중에 하나가 링크라고 배운 관계로, 어설프게라도 링크를 넣어봅니다.

그러면 응답을 위한 별도의 클래스를 추가로 만들어줘야겠습니다. 그전에 테스트 부터 작성합니다.

## 테스트 및 구현1

### 유효한 page 와 size를 지정하여 요청한 경우

```java
@Test
@DisplayName("page와 size가 지정된 게시물을 요청하여 최근 게시물 순으로 정렬된 게시글 목록 수신")
void shouldReturnAPageOfPosts() {

    postRepository.saveAll(getNewPosts(10));

    ResponseEntity<PostsList> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=3&size=3", PostsList.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());
    Number totalPages = documentContext.read("$.totalPages");
    assertThat(totalPages).isEqualTo(4);
    Number totalPosts = documentContext.read("$.totalPosts");
    assertThat(totalPosts).isEqualTo(10);
    Number currentPage = documentContext.read("$.totalPosts");
    assertThat(currentPage).isEqualTo(3);
    String firstPage = documentContext.read("$.firstPage");
    assertThat(firstPage).isEqualTo("/api/v1/posts?page=0&size=3");
    String lastPage = documentContext.read("$.lastPage");
    assertThat(lastPage).isEqualTo("/api/v1/posts?page=3&size=3");
    String previousPage = documentContext.read("$.previousPage");
    assertThat(previousPage).isEqualTo("/api/v1/posts?page=2&size=3");
    String nextPage = documentContext.read("$.nextPage");
    assertThat(nextPage).isNullOrEmpty();
    List<PostsList.PostInfo> postsInfo = documentContext.read("$.postsInfo");
    assertThat(postsInfo.size()).isEqualTo(1);
    PostsList.PostInfo postInfo = postsInfo.get(0);
    assertThat(postInfo.id()).isEqualTo(10);
    assertThat(postInfo.title()).isEqualTo("title10");
    assertThat(postInfo.writer()).isEqualTo("limvik@limvik.com");
    assertThat(postInfo.createdAt()).isBefore(LocalDateTime.now());
    assertThat(postInfo.uri()).isEqualTo("/api/v1/posts/10");

}

private List<Post> getNewPosts(int count) {
    List<Post> posts = new ArrayList<>();
    var user = User.builder().id(1L).build();

    for (int i = 0; i < count; ++i) {
        posts.add(Post.builder()
                .title("title" + i)
                .content("content" + i)
                .userId(user)
                .build());
    }

    return posts;
}
```

경우에 따른 테스트를 모두 작성한 후에 구현하지 않고 이번에는 하나의 테스트를 작성하고 확인하는 식으로 하려고 합니다. 모든 테스트를 다 작성하고 구현하면, 작업 단위가 커지는 느낌입니다.

응답용 클래스가 없어서  오류가 발생하므로, 먼저 PostsList를 구현하겠습니다.

### PostsList 구현

먼저 PostsList 를 구현해줍니다. 딱히 메서드가 필요하지 않고, 데이터만 저장하면 되므로 record를 사용합니다. 게시판용으로 요약된 Post에 대한 정보를 다른 곳에서 사용할 일은 없으므로 inner record 로 PostInfo도 생성해줍니다.

뭔가 가독성이 떨어지는 느낌인데... best practice가 뭔지 아직은 잘 모르겠습니다.

```java
package com.limvik.wantedpreonboardingbackend.form;


import java.time.LocalDateTime;
import java.util.List;

public record PostsList (
    long totalPages,
    long totalPosts,
    long currentPage,
    String firstPage,
    String lastPage,
    String previousPage,
    String nextPage,
    List<PostInfo> postsInfo) {
    public record PostInfo(long id, String title, String writer, LocalDateTime createdAt, String uri) {}
}
```

### 테스트 실행

테스트를 실행해보면 500 Error가 발생합니다.

XAuthAuthenticationConverter 에서 NullPointerException이 발생합니다. 현재 convert 메서드를 요청하는 곳에서는 NoSuchElementException 만 처리를 하고 있습니다. Optional 사용 미숙으로 ofNullable 을 사용해야 하는데, of를 사용해서 null 대입 시 NullPointerException 이 발생한 것입니다. 그래서 먼저 of 에서 ofNullable로 수정합니다.

```java
String jwt = Optional.ofNullable(request.getHeader("X-AUTH-TOKEN")).orElseThrow();
```

그리고 다시 테스트를 수행해보면 Spring Security로 인해서 403(Forbidden)이 반환됩니다.

```
expected: 200 OK
 but was: 403 FORBIDDEN
```

### Controller 구현

먼저 이전과 같이 무조건 200(OK)를 반환하는 Controller를 구현합니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList() {
    return ResponseEntity.ok().build();
}
```

### SecurityConfig 접근 권한 수정

post와 관련된 SecurityFilterChain 에 서 다음과 같이 `/api/v1/posts` 경로에 대해 `GET` 메서드로 요청하는 경우는 모두 허용합니다.

```java
@Bean
public SecurityFilterChain postFilterChain(HttpSecurity http) throws Exception{
    return http
            .securityMatcher("/api/v1/posts/**")
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/posts", "POST")).hasRole("USER");
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/posts", "GET")).permitAll();})
            .csrf(AbstractHttpConfigurer::disable)
            .logout(AbstractHttpConfigurer::disable)
            .anonymous(AbstractHttpConfigurer::disable)
            .sessionManagement(config -> config.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterAfter(new XAuthAuthenticationFilter(getJwtProviderManager()),
                    RequestCacheAwareFilter.class)
            .build();
}
```

### 다시 테스트

다시 테스트를 해보면 반환되는 값이 없으므로 200 이 반환되는지 여부에 대한 테스트는 성공하지만, JSON을 parsing 할게 없어서 예외가 던져집니다.

```
java.lang.IllegalArgumentException: json object can not be null
```

### Controller 추가 구현

먼저 Controller 에 Service에서 page 를 받아와서 반환하는 구현을 추가합니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(Pageable pageable) {
    PostsList postsList = postService.getPage(pageable);
    return ResponseEntity.ok(postsList);
}
```

### Service 구현

링크를 넣은게 좀 오버였나 싶기도 한데, 프론트 작업할 때 편하겠죠...?

```java
package com.limvik.wantedpreonboardingbackend.service;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import com.limvik.wantedpreonboardingbackend.domain.User;
import com.limvik.wantedpreonboardingbackend.form.PostsList;
import com.limvik.wantedpreonboardingbackend.repository.PostRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;

@RequiredArgsConstructor
@Service
public class PostService {

    private final PostRepository postRepository;

    @Transactional
    public Post create(Post newPost, User user) {
        newPost.setUserId(user);
        return postRepository.save(newPost);
    }

    @Transactional(readOnly = true)
    public PostsList getPage(Pageable pageable) {
        Page<Post> postsPage = postRepository.findAll(PageRequest.of(
                pageable.getPageNumber(),
                pageable.getPageSize(),
                pageable.getSortOr(Sort.by(Sort.Direction.DESC, "id"))
        ));

        return getPostsList(postsPage);
    }

    private PostsList getPostsList(Page<Post> postsPage) {
        Pageable pageable = postsPage.getPageable();
        int pageSize = pageable.getPageSize();
        int totalPages = postsPage.getTotalPages();
        URI firstPage = buildPostsListUri(0, pageSize);
        URI lastPage = buildPostsListUri(totalPages - 1, pageSize);
        URI previousPage = postsPage.isFirst() ?
                null : buildPostsListUri(pageable.getPageNumber() - 1, pageSize);
        URI nextPage = postsPage.isLast() ?
                null : buildPostsListUri(pageable.getPageNumber() + 1, pageSize);

        return new PostsList(
                postsPage.getTotalPages(),
                postsPage.getTotalElements(),
                pageable.getPageNumber(),
                firstPage,
                lastPage,
                previousPage,
                nextPage,
                convertToPostInfoFrom(postsPage)
                );
    }

    private URI buildPostsListUri(long page, long size) {
        return UriComponentsBuilder.newInstance()
                .path("/api/v1/posts")
                .queryParam("page", page)
                .queryParam("size", size)
                .build()
                .toUri();
    }

    private List<PostsList.PostInfo> convertToPostInfoFrom(Page<Post> postsPage) {
        List<PostsList.PostInfo> postsInfo = new ArrayList<>();
        for (var post : postsPage.getContent()) {
            postsInfo.add(new PostsList.PostInfo(
                    post.getId(),
                    post.getTitle(),
                    post.getUserId().getEmail(),
                    post.getCreatedAt(),
                    buildPostUri(post)
            ));
        }
        return postsInfo;
    }

    private URI buildPostUri(Post post) {
        return UriComponentsBuilder.newInstance()
                .path("/api/v1/posts/{id}")
                .buildAndExpand(post.getId())
                .toUri();
    }
}
```

### PostsList 수정

다음으로 Service 에 표시했듯, PostsList에 저장되는 링크는 String에서 URI 로 변경했습니다. 또 Page 객체에서 총 페이지 수(totalPages)와 Pageable 객체의 현재 페이지 번호(currentPage)는 int로 반환하여 long으로 받을 일은 없기 때문에 long에서 int로 수정했습니다.

```java
package com.limvik.wantedpreonboardingbackend.form;


import java.net.URI;
import java.time.LocalDateTime;
import java.util.List;

public record PostsList (
    int totalPages,
    long totalPosts,
    int currentPage,
    URI firstPage,
    URI lastPage,
    URI previousPage,
    URI nextPage,
    List<PostInfo> postsInfo) {
    public record PostInfo(long id, String title, String writer, LocalDateTime createdAt, URI uri) {}
}
```

### 테스트 실행 및 수정

테스트 실행하면 오류가 나는데 굳이 글에 추가하지는 않겠습니다.

수정해야 할 항목이 많습니다.

1. 테스트를 작성할 때 마지막 페이지의 마지막 게시물 목록을 조회하는 테스트를 작성했습니다. 마지막에 있는 게시물을 확인하려면 가장 처음 게시물인 1번 게시물을 검사해야 하는데, 가장 최근 게시물인 10번 게시물 테스트를 작성해서 1번 게시물 테스트로 수정했습니다.
2. ReponseEntity에 PostsList 를 generic으로 설정했는데, JsonPath.parse() 메서드에는 요청에 대한 응답이 문자열 형태로 들어가야 하므로, ResponseEntity는 generic으로 String을 설정해야 해서 수정했습니다.
3. 링크를 URI로 변경함에 따라 이에 맞추어 변경했습니다. 또한 경로와 쿼리 파라미터가 맞는지 확인하려면 번거로운 작업이 많고, isEqualTo 는 호스트 주소를 검사하지 못하므로 contains 로 변경하였습니다.
4. 클래스(PostsList) 내의 클래스(PostInfo) 필드는 Map 형태로 반환되는 것을 테스트 코드 작성할 때 반영하지 않아 수정하였습니다.

```java
@Test
@DisplayName("page와 size가 지정된 게시물을 요청하여 최근 게시물 순으로 정렬된 게시글 목록 수신")
void shouldReturnAPageOfPosts() {

    postRepository.saveAll(getNewPosts(10));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=3&size=3", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    Number totalPages = documentContext.read("$.totalPages");
    assertThat(totalPages).isEqualTo(4);
    Number totalPosts = documentContext.read("$.totalPosts");
    assertThat(totalPosts).isEqualTo(10);
    Number currentPage = documentContext.read("$.currentPage");
    assertThat(currentPage).isEqualTo(3);
    URI firstPage = URI.create(documentContext.read("$.firstPage"));
    assertThat(firstPage.toString()).contains("/api/v1/posts?page=0&size=3");
    URI lastPage = URI.create(documentContext.read("$.lastPage"));
    assertThat(lastPage.toString()).contains("/api/v1/posts?page=3&size=3");
    URI previousPage = URI.create(documentContext.read("$.previousPage"));
    assertThat(previousPage.toString()).contains("/api/v1/posts?page=2&size=3");
    String nextPage = documentContext.read("$.nextPage");
    assertThat(nextPage).isNullOrEmpty();
    List<Map<String, Object>> postsInfo = documentContext.read("$.postsInfo");
    assertThat(postsInfo.size()).isEqualTo(1);
    postsInfo.forEach(e -> {
        assertThat(e.get("id")).isEqualTo(1);
        assertThat(e.get("title")).isEqualTo("title1");
        assertThat(e.get("writer")).isEqualTo("limvik@limvik.com");
        assertThat(LocalDateTime.parse(String.valueOf(e.get("createdAt")))).isBefore(LocalDateTime.now());
        assertThat(e.get("uri").toString()).contains("/api/v1/posts/1");
    });
}
```

### page와 size를 지정하지 않고 게시물 목록 요청 테스트

새로운 구현을 해야하는 테스트는 아니라 곁다리로 추가합니다. Spring Data 에서 지원하는 Paging 관련 기능의 기본 값은 page 는 0, size 는 20 입니다. 저는 아직 익숙하지 않고, 추후에 기본값을 바꾼다면 정상적으로 동작하는지 테스트하기 위해 테스트를 추가합니다.

```java
@Test
@DisplayName("page와 size를 지정하지 않고 게시물 목록을 요청")
void shouldReturnPostsWithDefaultPageAndSize() {
    postRepository.saveAll(getNewPosts(21));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    Number totalPages = documentContext.read("$.totalPages");
    assertThat(totalPages).isEqualTo(2);
    Number totalPosts = documentContext.read("$.totalPosts");
    assertThat(totalPosts).isEqualTo(21);
    Number currentPage = documentContext.read("$.currentPage");
    assertThat(currentPage).isEqualTo(0);
    URI firstPage = URI.create(documentContext.read("$.firstPage"));
    assertThat(firstPage.toString()).contains("/api/v1/posts?page=0&size=20");
    URI lastPage = URI.create(documentContext.read("$.lastPage"));
    assertThat(lastPage.toString()).contains("/api/v1/posts?page=1&size=20");
    String previousPage = documentContext.read("$.previousPage");
    assertThat(previousPage).isNullOrEmpty();
    String nextPage = URI.create(documentContext.read("$.nextPage")).toString();
    assertThat(nextPage).contains("/api/v1/posts?page=1&size=20");
    List<Map<String, Object>> postsInfo = documentContext.read("$.postsInfo");
    assertThat(postsInfo.size()).isEqualTo(20);
    assertThat(postsInfo.get(0).get("id")).isEqualTo(21);
    assertThat(postsInfo.get(0).get("title")).isEqualTo("title21");
    assertThat(postsInfo.get(0).get("writer")).isEqualTo("limvik@limvik.com");
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get(0).get("createdAt")))).isBefore(LocalDateTime.now());
    assertThat(postsInfo.get(0).get("uri").toString()).contains("/api/v1/posts/21");
}
```

테스트를 수행해보면 모두 PASSED가 됩니다.

```
PostRequestTests > page와 size를 지정하지 않고 게시글 목록 요청 PASSED
PostRequestTests > 유효한 page와 size를 지정하여 게시글 목록 요청 PASSED
```

### 리팩터링

앞선 테스트 Display Name이 너무 길어서 짧게 줄이고, 게시`글`과 게시`물`을 혼용하고 있어서 게시`글`로 통일했습니다.

```
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > page와 size를 지정하지 않고 게시글 목록 요청 PASSED
PostRequestTests > 만료된 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > 게시글 목록이 없는 상태에서 요청 PASSED
PostRequestTests > 유효한 page와 size를 지정하여 게시글 목록 요청 PASSED
PostRequestTests > 유효한 토큰으로 `제목`이 비어있는 게시글 생성 요청 PASSED
PostRequestTests > 유효한 토큰으로 `내용`이 비어있는 게시글 생성 요청 PASSED
```

## 테스트 및 구현2

이번에는 숫자이기는 하지만 범위를 벗어나는 값을 page 와 size에 입력한 경우를 확인합니다.

### 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청

테스트 케이스를 만들어보자면 아래와 같이 작성할 수 있습니다. 이때 `size 정상`은 기본값으로 설정하여 기본값이 반환될 때의 값과 맞춥니다. 테스트 코드 좀 줄일겸...

||  | page | size |
|--|--|--|--|
|1| page 음수, size 정상 | -999 | 20 |
|2| page 정상, size 음수 | 0 | -999 |
|3| page 정상, size 0 | 0 | 0 |
|4| page 초과, size 정상 | 999 | 20 |
|5| 둘 다 비정상 | 999 | 0 |
|6| 둘 다 정상 | 0 | 20 |

그리고 ParameterizedTest 를 써보고 싶었는데 이번에 써봅니다.

```java
@ParameterizedTest
@CsvSource({"-999, 20", "0, -999", "0, 0", "999, 20", "999, 0", "0, 20"})
@DisplayName("범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청")
void shouldNotReturnPostsWithDefaultPageAndSize(int page, int size) {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=%d&size=%d".formatted(page, size), String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    Number totalPages = documentContext.read("$.totalPages");
    assertThat(totalPages).isEqualTo(1);
    Number totalPosts = documentContext.read("$.totalPosts");
    assertThat(totalPosts).isEqualTo(1);
    Number currentPage = documentContext.read("$.currentPage");
    assertThat(currentPage).isEqualTo(0);
    URI firstPage = URI.create(documentContext.read("$.firstPage"));
    assertThat(firstPage.toString()).contains("/api/v1/posts?page=0&size=20");
    URI lastPage = URI.create(documentContext.read("$.lastPage"));
    assertThat(lastPage.toString()).contains("/api/v1/posts?page=0&size=20");

}
```

그리고 테스트를 수행해보면 page 값을 초과하는 4번과 5번만 테스트를 통과하지 못합니다.

```
[1] page=-999, size=20 PASSED
[2] page=0, size=-999 PASSED
[3] page=0, size=0 PASSED
[4] page=999, size=20 FAILED
[5] page=999, size=0 FAILED
[6] page=0, size=20 PASSED
```

정상적인 응답(200, OK)이 오기는 했는데, 현재 페이지가 999 로 반환됩니다.

```
expected: 0
 but was: 999
```

그럼 Service 에서 currentPage 값을 조건에 따라 달라지게 수정해야겠습니다.

### PostService 수정

다음과 같이 입력받을 page 값이 전체 페이지 수와 비교하여 현재 페이지(currentPage) 값을 지정하도록 수정합니다.

```java
int currentPage = totalPages >= pageable.getPageNumber() ? pageable.getPageNumber() : 0;
```

### 테스트

다른 테스트도 같이 해보면, 이상 없이 PASSED 되는 것을 볼 수 있습니다.

```
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > page와 size를 지정하지 않고 게시글 목록 요청 PASSED
PostRequestTests > 만료된 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > 게시글 목록이 없는 상태에서 요청 PASSED
PostRequestTests > 유효한 page와 size를 지정하여 게시글 목록 요청 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [1] page=-999, size=20 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [2] page=0, size=-999 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [3] page=0, size=0 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [4] page=999, size=20 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [5] page=999, size=0 PASSED
PostRequestTests > 범위를 벗어나는 page와 size를 지정하여 게시글 목록 요청 > [6] page=0, size=20 PASSED
PostRequestTests > 유효한 토큰으로 `제목`이 비어있는 게시글 생성 요청 PASSED
PostRequestTests > 유효한 토큰으로 `내용`이 비어있는 게시글 생성 요청 PASSED
```

그리고 생각이 납니다. 정상인 경우를 왜 테스트 했지...?

삭제

```java
@CsvSource({"-999, 20", "0, -999", "0, 0", "999, 20", "999, 0"})
```

## 테스트 및 구현3

이번엔 문자 혹은 문자열을 page 또는 size에 입력한 경우에 대한 테스트를 작성합니다. 숫자가 범위를 벗어날 때는 실수하는 경우가 될 수도 있지만, 문자는 그냥 고의적이고 게시글을 조회할 목적이 아닌 것이라 판단하여 앞서 정했던대로 400(Bad Request) 를 반환하겠습니다.

### page 또는 size에 문자를 입력하여 게시글 목록 요청

```java
@Test
@DisplayName("page 또는 size에 문자를 입력하여 게시글 목록 요청")
void shouldNotReturnPostsIfNotANumber() {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=char&size=string", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
}
```

그리고 테스트를 해보면, 생각지도 못한 200(OK)가 나옵니다.

```
expected: 400 BAD_REQUEST
 but was: 200 OK
```

Controller 측에서만 수정하면 되니, 어려움은 없겠습니다.

### Controller 수정

문자열을 입력했을 때 Pageable 에 저장되어 전달되는 page 와 size의 값을 보니 기본값입니다.

![디버거를 이용하여 살펴본 Pageable 내 page, size 값](/assets/img/2023-08-13-making-rest-api-with-spring-boot-4-request-posts-list/03-page-size-are-char.png)

그렇다면, request 에서 쿼리 파라미터 값을 가져와야겠습니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(HttpServletRequest request, Pageable pageable) {
    try {
        Integer.parseInt(request.getParameter("page"));
        Integer.parseInt(request.getParameter("size"));
    } catch (NumberFormatException e) {
        return ResponseEntity.badRequest().build();
    }

    PostsList postsList = postService.getPage(pageable);
    return ResponseEntity.ok(postsList);
}
```

숫자로 변환 시에 NumberFormatException 예외가 던져지면 400(Bad Request)를 반환합니다.

차후에 page 나 size 입력값을 제한하는데도 사용할 수 있겠습니다.

테스트를 수행해보면, PASSED.

```
PostRequestTests > page 또는 size에 문자를 입력하여 게시글 목록 요청 PASSED
```

## 테스트 및 구현4

이번에는 정렬과 관련한 테스트를 구현합니다. 먼저, 정상적으로 컬럼과 정렬 방향을 지정해서 정렬 요청하는 것을 하겠습니다.

### 유효한 page와 size 그리고 유효한 sort 값을 이용한 게시글 목록 요청

간단하게 기본값인 desc(내림차순) 이 아닌 asc(오름차순)으로 변경하여 게시글 목록을 요청하는 테스트를 작성합니다.

```java
@Test
@DisplayName("유효한 page와 size 그리고 유효한 sort 값을 이용한 게시글 목록 요청")
void shouldReturnPostsWithCustomSort() {
    postRepository.saveAll(getNewPosts(5));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=0&size=2&sort=id,asc", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

    DocumentContext documentContext = JsonPath.parse(createResponse.getBody());

    Number totalPages = documentContext.read("$.totalPages");
    assertThat(totalPages).isEqualTo(3);
    Number totalPosts = documentContext.read("$.totalPosts");
    assertThat(totalPosts).isEqualTo(5);
    Number currentPage = documentContext.read("$.currentPage");
    assertThat(currentPage).isEqualTo(0);
    URI firstPage = URI.create(documentContext.read("$.firstPage"));
    assertThat(firstPage.toString()).contains("/api/v1/posts?page=0&size=2");
    URI lastPage = URI.create(documentContext.read("$.lastPage"));
    assertThat(lastPage.toString()).contains("/api/v1/posts?page=2&size=2");
    String previousPage = documentContext.read("$.previousPage");
    assertThat(previousPage).isNullOrEmpty();
    String nextPage = URI.create(documentContext.read("$.nextPage")).toString();
    assertThat(nextPage).contains("/api/v1/posts?page=1&size=2");
    List<Map<String, Object>> postsInfo = documentContext.read("$.postsInfo");
    assertThat(postsInfo.size()).isEqualTo(2);
    assertThat(postsInfo.get(0).get("id")).isEqualTo(1);
    assertThat(postsInfo.get(0).get("title")).isEqualTo("title1");
    assertThat(postsInfo.get(0).get("writer")).isEqualTo("limvik@limvik.com");
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get(0).get("createdAt")))).isBefore(LocalDateTime.now());
    assertThat(postsInfo.get(0).get("uri").toString()).contains("/api/v1/posts/1");
}
```

테스트는 통과하고, 구현할게 없습니다.

```
유효한 page와 size 그리고 유효한 sort 값을 이용한 게시글 목록 요청 PASSED
```

### 유효한 page와 size 그리고 잘못된 sort 값을 이용한 게시글 목록 요청

유효하지 않은 컬럼 값을 이용한 경우와 유효하지 않은 정렬 값을 이용해서 게시글 목록을 요청하는 테스트를 작성해보겠습니다. 이때는 page 또는 size에 문자열을 입력했을 때와 마찬가지로 400(Bad Request)를 반환하도록 하겠습니다. sort 값을 잘못 입력했을 때, 기본값을 이용해서 반환하면 정상적으로 데이터를 받았다고 착각할 가능성도 있기 때문입니다.

```java
@ParameterizedTest
@CsvSource({"limvik, asc", "id, limvik"})
@DisplayName("유효한 page와 size 그리고 잘못된 sort 값을 이용한 게시글 목록 요청")
void shouldNotReturnPostsIfInvalidSort(String sortBy, String order) {
    postRepository.saveAll(getNewPosts(1));

    ResponseEntity<String> createResponse =
            restTemplate.getForEntity("/api/v1/posts?page=0&size=2&sort=%s,%s".formatted(sortBy, order), String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
}
```

그리고 테스트를 해보면, 이번에는 500 이 반환됩니다.

```
expected: 400 BAD_REQUEST
 but was: 500 INTERNAL_SERVER_ERROR
```

이번에도 유연하게 넘어갈 줄 알았더니 아닙니다.

### Controller 수정하기

Pageable 에 정렬과 관련된 값을 검증하도록 수정합니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(HttpServletRequest request, Pageable pageable) {
    try {
        hasNumberFormatParameter(request);
        hasValidSortPropertyAndOrder(pageable);
    } catch (Exception e) {
        return ResponseEntity.badRequest().build();
    }

    PostsList postsList = postService.getPage(pageable);
    return ResponseEntity.ok(postsList);
}

void hasNumberFormatParameter(HttpServletRequest request) throws NumberFormatException {
    List.of(request.getParameter("page"), request.getParameter("size")).forEach(Integer::parseInt);
}

void hasValidSortPropertyAndOrder(Pageable pageable) throws NoSuchSortException {

    int countSortProperties = (int) pageable.getSort().get().count();

    AtomicInteger countMatch = new AtomicInteger();
    Arrays.stream(Post.class.getDeclaredFields()).forEach(field -> {
        if (pageable.getSort().stream().anyMatch(order ->
                order.getProperty().equals(field.getName()) &&
                        (order.isAscending() || order.isDescending())
        )) {
            countMatch.incrementAndGet();
        }
    });

    if (countSortProperties != countMatch.get())
        throw new NoSuchSortException();
}
```

비교할게 많아서 좀 복잡해졌습니다. Sorting 시에 여러 컬럼을 지정할 수 있으므로, 혹시라도 여러 컬럼을 지정해서 입력이 들어올 때를 대비해 모두 정상적인 정렬 값인지 확인하도록 하였습니다.

재밌는건 컬럼 값만 일치하면 asc(오름차순)가 기본값이 되어 정상적인 Sort 값이 새로 생깁니다. ParameterizedTest 의 두 번째 입력값 `"id, limvik"` 의 컬럼 이름은 정상이라서 그런지 디버거로 보면 `id: asc` 가 하나 더 생겼음을 확인할 수 있습니다.

![디버거를 이용해서 본 잘못된 Sorting 값 테스트 과정](/assets/img/2023-08-13-making-rest-api-with-spring-boot-4-request-posts-list/04-invalid-sort-test.png)

### NoSuchSortException 추가

Sort 값이 잘못 되었을 때를 위해 Exception 도 하나 만들어줬습니다.

```java
package com.limvik.wantedpreonboardingbackend.exception;

public class NoSuchSortException extends RuntimeException {
    public NoSuchSortException(String message) {
        super(message);
    }

    public NoSuchSortException() {
        this("유효하지 않은 정렬입니다.");
    }
}
```

### 테스트 수행

PASSED 가 시현되는 것을 볼 수 있습니다.

```
[1] sortBy=limvik, order=asc PASSED
[2] sortBy=id, order=limvik PASSED
```

### Controller 오류 수정 및 리팩터링

정수가 아닌 값으로 page, size 쿼리 파라미터를 지정했을 때 예외가 발생하여 400(Bad Request)가 반환되도록 하였습니다.

문자열인지 여부를 판단하기 위해 다음과 같이 코드를 작성했는데, null 일 때는 기본값을 이용해서 200(Ok)으로 응답해야하는데 여기서 예외가 발생합니다.

```java
void hasNumberFormatParameter(HttpServletRequest request) throws NumberFormatException {  
    List.of(request.getParameter("page"), request.getParameter("size")).forEach(Integer::parseInt);  
}
```

그래서 다음과 같이 null 일 때는 숫자 문자열 "0" 을 반환하도록 수정합니다.

```java
private void hasNumberFormatParameter(HttpServletRequest request) throws NumberFormatException {

    String[] parameters = {"page", "size"};

    for (var parameter : parameters)
        Integer.parseInt(Optional.ofNullable(request.getParameter(parameter)).orElse("0"));

}
```

다음으로 try catch 문은 함수로 따로 빼라는 밥 아저씨(클린 코드 저자)의 조언에 따라, private 메서드로 빼주었습니다. 또, 기존의 다른 코드도 가독성 면에서 안좋아 수정했습니다. 특히 Sort 값을 검증할 때 stream을 중첩되게 사용하여 보기 불편했는데, for 문으로 변경하니 가독성이 더 좋아졌습니다.

PostController 를 수정한 코드는 아래와 같습니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(HttpServletRequest request, Pageable pageable) {

    if (isBadRequest(request, pageable))
        return ResponseEntity.badRequest().build();

    PostsList postsList = postService.getPage(pageable);
    return ResponseEntity.ok(postsList);

}

private boolean isBadRequest(HttpServletRequest request, Pageable pageable) {

    try {
        hasNumberFormatParameter(request);
        hasValidSortPropertyAndOrder(pageable);
    } catch (NumberFormatException | NoSuchSortException e) {
        return true;
    }
    return false;

}

private void hasNumberFormatParameter(HttpServletRequest request) throws NumberFormatException {

    String[] parameters = {"page", "size"};

    for (var parameter : parameters)
        Integer.parseInt(Optional.ofNullable(request.getParameter(parameter)).orElse("0"));

}

private void hasValidSortPropertyAndOrder(Pageable pageable) throws NoSuchSortException {

    int countMatch = 0;

    for (var field : Post.class.getDeclaredFields()) {
        if (pageable.getSort().stream().anyMatch(order ->
                order.getProperty().equals(field.getName()) &&
                        (order.isAscending() || order.isDescending()))) {
            ++countMatch;
        }
    }

    if (pageable.getSort().get().count() != countMatch)
        throw new NoSuchSortException();

}
```

음... 근데 갑자기 Controller 에 이 코드가 위치하는게 맞는가... 에 대한 의문이 생깁니다. Controller, Service 의 중점적인 사항은 명확한데, 부수적인 처리는 코드를 어디에 배치해야할지 아직 혼란스럽습니다. 당장은 Controller 에서 요청 값에 대한 처리를 하고, Service 에서 반환할 값에 대한 처리를 하는걸로 해야겠습니다.

## 테스트 및 구현5

아직 끝나지 않았습니다. 게시글이 없다면 204(No Content) 를 반환해야 합니다.

### 게시글이 없는 상태에서 요청

게시글을 추가하지 않고, 요청했을 때 204(No Content) 를 반환하고, body 는 비어있는지 테스트하는 코드를 작성합니다.

```java
@Test
@DisplayName("게시글 목록이 없는 상태에서 요청")
void shouldReturnNoContentWithNoPost() {

    ResponseEntity<String> createResponse = restTemplate
            .getForEntity("/api/v1/posts", String.class);

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);
    assertThat(createResponse.getBody()).isNullOrEmpty();
    
}
```

테스트를 수행하면 200(OK)가 반환되어 실패합니다.

### Controller 수정

작성된 게시글이 없을 때는 Service에서 null을 반환하여 204(No Content)를 반환합니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(HttpServletRequest request, Pageable pageable) {

    if (isBadRequest(request, pageable))
        return ResponseEntity.badRequest().build();

    PostsList postsList = postService.getPage(pageable);
    if (postsList != null) {
        return ResponseEntity.ok(postsList);
    } else {
        return ResponseEntity.noContent().build();
    }

}
```

### Service 수정

Service 에서 게시글이 없을 경우 null을 반환하도록 수정합니다.

```java
@Transactional(readOnly = true)
public PostsList getPage(Pageable pageable) {
    Page<Post> postsPage = postRepository.findAll(PageRequest.of(
            pageable.getPageNumber(),
            pageable.getPageSize(),
            pageable.getSortOr(Sort.by(Sort.Direction.DESC, "id"))
    ));

    if (postsPage.getTotalElements() == 0) {
        return null;
    } else {
        return getPostsList(postsPage);
    }
}
```

### 다시 테스트 수행

```
게시글이 없는 상태에서 요청 PASSED
```

## 빌드 테스트 수행

마무리 점검 차 커맨드라인에서 `./gradew test` 를 입력하여 빌드 테스트를 하니 Post 클래스에 대한 Serialization 및 Deserialization 테스트가 실패합니다.

```
PostJsonTests > PostDeserializationTest() FAILED
PostJsonTests > PostSerializationTest() FAILED
```

이전에 @ManyToOne 관계를 설정했을 때 관련한 더미데이터와 코드들을 다 수정해주지 않아서 문제가 발생했습니다.

### Post, User 클래스의 관계명과 관련된 코드 수정

이 기회에 Post 클래스에서 관계를 설정한 필드 내 User 클래스의 이름을 userId 에서 user로 변경하고 테이블 생성 시 컬럼명만 user_id가 되도록 수정하겠습니다.

```java
// Post.java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id")
private User user;

// User.java
@JsonIgnore
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
List<Post> posts;
```

이걸 설정하면서 mappedBy에 Post 에서 설정한 JoinColumn 의 name 과 동일하게 설정해야하는 것으로 제가 잘못알고 있었단 것을 알게됐습니다. `mappedBy 에는 관계를 설정한 클래스의 필드명과 동일한 이름을 작성`해야합니다.

이외에도 Post 클래스를 이용하면서 getUserId() 를 했던 것을 getUser() 로 변경합니다.

### 다시 테스트 수행

다시 테스트를 수행해보면 모두 성공하는 것을 볼 수 있습니다.

```
PostJsonTests > PostDeserializationTest() PASSED
PostJsonTests > PostSerializationTest() PASSED
PostRequestTests > page 또는 size에 문자를 입력하여 게시글 목록 요청 PASSED
PostRequestTests > 유효한 page와 size 그리고 유효한 sort 값을 이용한 게시글 목록 요청 PASSED
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > page와 size를 지정하지 않고 게시글 목록 요청 PASSED
PostRequestTests > 범위를 벗어나는 page 또는 size를 지정하여 게시글 목록 요청 > [1] page=-999, size=20 PASSED
PostRequestTests > 범위를 벗어나는 page 또는 size를 지정하여 게시글 목록 요청 > [2] page=0, size=-999 PASSED
PostRequestTests > 범위를 벗어나는 page 또는 size를 지정하여 게시글 목록 요청 > [3] page=0, size=0 PASSED
PostRequestTests > 범위를 벗어나는 page 또는 size를 지정하여 게시글 목록 요청 > [4] page=999, size=20 PASSED
PostRequestTests > 범위를 벗어나는 page 또는 size를 지정하여 게시글 목록 요청 > [5] page=999, size=0 PASSED
PostRequestTests > 만료된 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > 게시글이 없는 상태에서 요청 PASSED
PostRequestTests > 유효한 page와 size 그리고 잘못된 sort 값을 이용한 게시글 목록 요청 > [1] sortBy=limvik, order=asc PASSED
PostRequestTests > 유효한 page와 size 그리고 잘못된 sort 값을 이용한 게시글 목록 요청 > [2] sortBy=id, order=limvik PASSED
PostRequestTests > 유효한 page와 size를 지정하여 게시글 목록 요청 PASSED
PostRequestTests > 유효한 토큰으로 `제목`이 비어있는 게시글 생성 요청 PASSED
PostRequestTests > 유효한 토큰으로 `내용`이 비어있는 게시글 생성 요청 PASSED
UserJsonTests > userAccountDeserializationTest() PASSED
UserJsonTests > userAccountSerializationTest() PASSED
UserLoginTests > shouldNotCreateANewTokenIfWrongPassword() PASSED
UserLoginTests > shouldNotCreateANewTokenIfUserDoesNotExist() PASSED
UserRequestTests > shouldCreateANewUserAccount() PASSED
UserRequestTests > shouldNotCreateANewTokenIfUserDoesNotExist() SKIPPED
UserRequestTests > shouldCreateANewTokenIfUserExist() PASSED
UserRequestTests > shouldNotCreateANewUserAccountIfNotValid() PASSED

BUILD SUCCESSFUL in 18s
```

토종 한국인이라 아직 DisplayName을 설정하지 않은 테스트에 이름을 달아주긴 해야겠습니다.

## 기능 테스트

이제 Postman 을 이용해서 기능이 제대로 동작하는지 테스트 해봐야합니다.

그런데 매번 회원가입하고 테스트용 게시글 만드는게 귀찮으니까, 더미 데이터를 실행 시에 자동으로 추가하도록 설정합니다.

### 더미데이터용 비밀번호 생성 및 회원 정보 삽입 SQL 생성

Postman 에서 회원가입 api로 요청을 보내고, 데이터베이스를 조회해서 비밀번호를 가져와서 INSERT 문을 작성합니다.

```sql
INSERT INTO users (created_at, email, password) VALUES ('2023-07-26 10:30:17', 'limvik@limvik.com', '$2a$10$LMvk.4ul/ZPS5t/8shZAW.pGnS0downROxBWSWqk/J645R.63G7XG');
```

### 더미 게시글 생성

직접 작성하기는 귀찮으니까 GPT 에게 부탁합니다. 공손하게...

```sql
INSERT INTO posts (created_at, updated_at, user_id, title, content)
VALUES
    ('2023-08-10 10:12:34', '2023-08-10 10:12:34', 1, '취업 프로세스에 대한 질문', '안녕하세요. 면접에서 어떻게 준비해야 하는지 궁금합니다.'),
    ('2023-08-09 15:15:00', '2023-08-09 15:15:00', 2, '이력서 작성 팁 공유합니다.', '이력서에 성과 위주로 작성하는 것이 좋다는 경험을 했습니다. 여러분도 도전해보세요.'),
    ('2023-08-09 13:40:12', '2023-08-09 13:40:12', 3, '기술 면접 대비 방법', '기술 면접에서는 기본적인 CS 지식과 관련 프레임워크에 대한 지식이 필요하다고 느꼈습니다.'),
    ...
```

총 30개를 요청해서 만들었습니다.

### 더미데이터 파일 생성

`resources` 디렉터리 밑에 `data.sql` 파일을 만들고 위에서 작성한 sql 을 모두 붙여넣습니다.

### application.properties 수정

application.properties 에 다음과 같은 속성을 추가합니다.

```properties
spring.jpa.defer-datasource-initialization=true  
spring.sql.init.mode=always
```

`spring.jpa.defer-datasource-initialization` 은 JPA의 EntityManagerFactory bean 을 초기화 한 후에 데이터 소스(데이터베이스)를 `초기화(initialization)`할지 여부를 선택하는 설정입니다.

그리고 `spring.sql.init.mode` 는 data.sql 에 작성한 sql을 실행(`초기화`)시키는 설정을 할 수 있는 설정입니다.

이렇게 설정하면 테스트에서도 sql이 실행되므로, 테스트 디렉토리에 있는 application.properties 에는 always가 아닌 `never` 로 설정해야합니다.

### 실행

간단하게 쿼리 파라미터 없이 GET 요청을 했는데, 링크에 host가 빠져있습니다.

![Postman 에서 쿼리 파라미터 없이 게시물 목록 요청](/assets/img/2023-08-13-making-rest-api-with-spring-boot-4-request-posts-list/05-request-posts-list-in-postman.png)

UriComponentsBuilder 넘기기 귀찮아서 새로운 인스턴스를 만들었더니, host 데이터가 빠졌나봅니다.

귀찮지만 ucb를 Controller 에서 받아서 Service 로 넘겨주어 같은 UriComponentsBuilder 인스턴스로 uri를 생성하도록 수정합니다.

간단하게 수정한 Controller 소스만 첨부합니다.

```java
@GetMapping
public ResponseEntity<PostsList> getPostsList(HttpServletRequest request,
                                              Pageable pageable,
                                              UriComponentsBuilder ucb) {

    if (isBadRequest(request, pageable))
        return ResponseEntity.badRequest().build();

    PostsList postsList = postService.getPage(pageable, ucb);
    if (postsList != null) {
        return ResponseEntity.ok(postsList);
    } else {
        return ResponseEntity.noContent().build();
    }

}
```

그리고 다시 요청을 보내보면, Host 도 함께 링크에 포함되어 있습니다.

![Uri 관련 코드 수정 후 Postman 에서 다시 요청을 보낸 결과 화면](/assets/img/2023-08-13-making-rest-api-with-spring-boot-4-request-posts-list/06-request-posts-list-in-postman-after-modify-uri.png)

이정도로 마무리하고 나머지 기능도 작업해야겠습니다.

## 추가 수정

Outro 까지 다 작성했는데, PR 올리면서 Postman 으로 전체적으로 다시 테스트 했더니 예상과 다른 동작이 있습니다.

### 개별 게시글 링크에 다음 페이지의 page 값이 지정되는 문제

개별 게시물의 uri 를 build 할 때는 page 나 size 를 지정하지 않다보니 이전에 마지막으로 지정했던 다음 페이지의 page와 size 값을 그대로 사용하는 오류가 있었습니다.

그래서 다음 페이지의 uri를 build 한 후 다시 현재 페이지의 값으로 변경하도록 코드를 추가하였습니다.

```java
URI nextPage = postsPage.isLast() ?
        null : buildPostsListUri(currentPage + 1, pageSize, ucb);
ucb.replaceQueryParam("page", currentPage)
        .replaceQueryParam("size", pageSize)
        .build();
```

size는 동일하니 빼도 되는데, 같이 변경을 해버렸네요.

### page 범위를 초과하는 경우 개별 게시글 목록 반환하지 않는 문제

page 범위를 초과하는 값을 입력 받았을 때, 해당 값을 그대로 repository 에서 조회를 해서 반환값이 비어있었습니다.

page 범위 초과 테스트할 때 body 에 대한 테스트를 작성하지 않았더니 여기서 바로 당합니다.

전체 페이지를 계산해서 입력된 page 값과 비교하여 실제 총 page 값보다 큰 경우 page=0으로 조회하도록 수정합니다.

```java
@Transactional(readOnly = true)
public PostsList getPage(Pageable pageable, UriComponentsBuilder ucb) {
    Page<Post> postsPage = postRepository.findAll(PageRequest.of(
            pageable.getPageNumber() > getTotalPages(pageable) ? 0 : pageable.getPageNumber(),
            pageable.getPageSize(),
            pageable.getSortOr(Sort.by(Sort.Direction.DESC, "id"))
    ));

    if (postsPage.getTotalElements() == 0) {
        return null;
    } else {
        return getPostsList(postsPage, ucb);
    }
}

private int getTotalPages(Pageable pageable) {
    long totalPosts = postRepository.count();
    int requestedPageSize = pageable.getPageSize();
    int totalPages = (int) (totalPosts / requestedPageSize);
    totalPages += totalPosts % requestedPageSize != 0 ? 1 : 0;
    return totalPages;
}
```

테스트를 먼저 작성했어야 하는데, 급한 마음에 구현부터 해버렸습니다. 간단하게 테스트 데이터인 page 가 999 일 때, 게시글 관련 정보가 포함되어 있는지 확인하는 테스트를 추가합니다.

```java
if (page == 999) {
    List<Map<String, Object>> postsInfo = documentContext.read("$.postsInfo");
    assertThat(postsInfo.size()).isEqualTo(1);
    assertThat(postsInfo.get(0).get("id")).isEqualTo(1);
    assertThat(postsInfo.get(0).get("title")).isEqualTo("title1");
    assertThat(postsInfo.get(0).get("writer")).isEqualTo("limvik@limvik.com");
    assertThat(LocalDateTime.parse(String.valueOf(postsInfo.get(0).get("createdAt")))).isBefore(LocalDateTime.now());
    assertThat(postsInfo.get(0).get("uri").toString()).contains("/api/v1/posts/1");
}
```

다시 테스트를 해보면 모두 PASSED가 되는 것을 확인할 수 있습니다. 붙여넣지는 않겠습니다.

size 도 무한정으로 하면 안될텐데, 추후에 page 와 size 기본값을 변경할 때 최대값 제한도 같이 해야겠습니다.

## Outro

page 또는 size 값이 범위를 벗어났을 때도, 어차피 원하는 값이 아니니까 Bad Request라 하는게 나으려나... 갑자기 이런 생각도 듭니다. 요구사항이 애매하고 물어볼데도 없으니 혼란의 연속입니다.

주말 중에는 완성하려 했는데, 과연 일요일에는 기능만이라도 완성이 가능할지 모르겠습니다.

밑에는 여기서 깊게 파기에는 너무 벗어나는 것 같아서 따로 다루려고 작성하다가 만 내용들입니다.

## Spring Data JPA의 saveAll() 메서드와 Transaction

SimpleJpaRepository 를 살펴보니 그냥 Iterable 객체를 가져와서 하나씩 save를 수행하는 방식이었습니다.


```java
@Transactional
@Override
public <S extends T> List<S> saveAll(Iterable<S> entities) {

    Assert.notNull(entities, "Entities must not be null");

    List<S> result = new ArrayList<>();

    for (S entity : entities) {
        result.add(save(entity));
    }

    return result;
}
```

Java 관련 검색을 하면 항상 같이 나오는 `Baeldung` 의 save와 saveAll 비교한 글([링크](https://www.baeldung.com/spring-data-save-saveall))에 의하면 saveAll 은 모두 하나의 트랜잭션 안에서 처리해서 성능이 더 좋다고 합니다. saveAll 은 다시 save를 호출하는데, @Transactional 을 표시한 메서드 안에서 또 @Transactional 로 표시된 메서드를 호출해도 Transaction이 하나로 유지되나봅니다.

다른 분의 글([링크](https://kim-solshar.tistory.com/71))을 따라 이에 대해 구체적으로 보자면, @Transactional 내부의 propagation 속성을 보면, 기본값이 REQUIRED로 되어있습니다.

```java
Propagation propagation() default Propagation.REQUIRED;
```

그리고 다시 Propagation.REQUIRED 를 보면 주석에 transaction이 있으면 그거 쓰고, 없으면 하나 만든다고 써있습니다.

```java
/**
 * Support a current transaction, create a new one if none exists.
 * Analogous to EJB transaction attribute of the same name.
 * <p>This is the default setting of a transaction annotation.
 */
REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
```

@Transactional 이 표시된 save 메서드를 @Transactional 이 표시된 saveAll 이 호출해도 하나의 transaction으로 처리될 수 있는 이유가 되겠습니다.

## IDENTITY generator 사용 시 INSERT 문 JDBC batching 비활성화 문제

page와 size를 지정해서 테스트하기 위해 먼저 10개의 post를 만들어 저장해줍니다.

```java
@Test
@DisplayName("page와 size가 지정된 게시물을 요청하여 최근 게시물 순으로 정렬된 게시글 목록 수신")
void shouldReturnAPageOfPosts() {

    postRepository.saveAll(getNewPosts(10));

}

private List<Post> getNewPosts(int count) {
    List<Post> posts = new ArrayList<>();
    var user = User.builder().id(1L).build();

    for (int i = 0; i < count; ++i) {
        posts.add(Post.builder()
                .title("title" + i)
                .content("content" + i)
                .userId(user)
                .build());
    }

    return posts;
}
```

saveAll 메서드를 처음 써봐서, 잘 되는지 확인해보기 위해 바로 테스트를 실행해봤습니다. 저장이 잘 되기는 하는데... Entity 갯수만큼 Insert 문을 작성하고 수행합니다.

![총 10번의 insert를 수행하는 테스트 로그 화면](/assets/img/2023-08-13-making-rest-api-with-spring-boot-4-request-posts-list/01-insert-by-saveall.png)

하나의 insert 문에 입력할 데이터만 이어붙이는 방식으로 SQL을 생성하여 모든 데이터를 저장해줄 것이라 생각했는데, 아니었습니다.

여러 자료들을 검토하고 따라해봤지만, 해결이 되지 않습니다. 지금 당장은 과제 완성이 더 중요한 우선순위라 기능과 직접적으로 관련이 없는 문제에 대한 해결은 다음에 하기 위해 참고자료를 다 붙여넣습니다.

https://stackoverflow.com/questions/50772230/how-to-do-bulk-multi-row-inserts-with-jparepository
https://kim-solshar.tistory.com/71
https://keitaroinc.medium.com/implementing-bulk-updates-with-spring-data-jpa-39e5a715783d
https://amrutprabhu.medium.com/spring-boot-jpa-bulk-insert-performance-by-100-times-14ec10fa682b
https://docs.jboss.org/hibernate/orm/6.0/userguide/html_single/Hibernate_User_Guide.html#best-practices-jdbc-batching

자료를 보다보니 제가 원하는 방식으로 insert를 수행하려면, 용어가 맞는지는 모르겠지만 JDBC batching 작업을 수행해야 합니다. 문제는 Post 클래스의 id는 `IDENTITY` generator를 사용하고 있는 것 입니다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

자세한 문제는 hibernate 공식 문서를 보면 나와있습니다. 아래와 같이 MySQL은 `IDENTITY` 를 사용할 수 밖에 없는데 `IDENTITY`를 설정하면 `INSERT` 문 실행 시 JDBC batching이 비활성화 된다고 언급하고 있습니다.

>If the underlying database supports sequences, you should always use them for your Hibernate entity identifiers.
>
>Only if the relational database does not support sequences (e.g. MySQL 5.7), you should use the  `IDENTITY`  generators. However, you should keep in mind that the  `IDENTITY`  generators disables JDBC batching for  `INSERT`  statements.

SEQUENCE 를 지원하면 SEQUENCE 를 쓰라는 꿀팁도...

위에 남긴 링크 중에 하나는 sequence table을 직접 하나 만들어서 지정해주는 방식을 사용하기도 합니다.

하지만 저는 h2 데이터베이스를 테스트에서 사용하고 있고, Post 클래스에서 SEQUENCE generator를 사용하면 아래와 같이 sequence 테이블을 생성하는 sql을 볼 수 있지만, 그대로 여러 INSERT 문이 수행되는 것을 볼 수 있었습니다.

```
Hibernate: 
    drop sequence if exists posts_seq
Hibernate: 
    create sequence posts_seq start with 1 increment by 50
```

현재 요구사항에는 테스트 외에는 batch insert 를 할 일이 없으니 다음에 별도의 글로 해결방법을 고민해보겠습니다.
