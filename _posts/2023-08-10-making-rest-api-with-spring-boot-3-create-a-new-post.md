---
layout: post
title: Spring Boot로 REST API 만들어보기 (3) 게시글 작성
categories:
- API
- REST
tags:
- Spring
- Spring Security
- JPA
- JWT
- JUnit5
date: 2023-08-10 23:39 +0900
---
## Intro

이전에는 로그인 엔드포인트를 추가([글 링크](https://limvik.github.io/posts/making-rest-api-with-spring-boot-2-2-login/))하였습니다. 이어서 다음 과제를 수행합니다.

역시나 이 글도 난장판이 될 예정이고, 생각해보니 코드 [링크](https://github.com/limvik/wanted-pre-onboarding-backend)를 걸기는했지만 코드도 난장판인 것 같습니다. 그래도 글 보다 보기 편하지 않을까... 싶은...?

## 요구사항

### 과제 3. 새로운 게시글을 생성하는 엔드포인트

이번에는 특별히 요구되는 사항은 없습니다.

## Post 클래스 생성하기

첫 글에서 테이블 설계([링크](https://limvik.github.io/posts/making-rest-api-with-spring-boot-1-signup/#%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%ED%85%8C%EC%9D%B4%EB%B8%94-%EC%84%A4%EA%B3%84))를 했던대로 Posts 클래스를 생성합니다.

![이전에 설계했던 데이터베이스의 ERD](/assets/img/2023-08-01-wanted-pre-onboarding-backend/01-Wanted-pre-onboarding-backend.svg)

JPA가 익숙하지 않아서 일대다 관계를 표현하는데 시간이 조금 걸리겠습니다.

```java
package com.limvik.wantedpreonboardingbackend.domain;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.time.LocalDateTime;


@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
@Entity
@Table(name = "posts")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    // MySQL 에서 LONGTEXT 로 지정되며, 최대 값은 4_294_967_295 bytes(4 GB) 입니다.
    @Lob
    @Column(nullable = false, length = 16_777_216)
    private String content;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user")
    private User user;

}

```

### @ManyToOne

과거에 JPA 예제 따라해봤던게 있어서 참고해서 작성했습니다. `FetchType.LAZY`는 관계가 직접 액세스되거나 조회될 때까지 해당 관계의 대상 엔터티를 로드하지 않는 지연 로딩을 사용하겠다는 것을 의미합니다. 요약하자면, Users 테이블에 있는 정보를 필요할 때만 불러오겠다는 의미입니다.

### LONGTEXT

MySQL의 경우 LONGTEXT로 지정이되도록 content의 length 를 16_777_216 로 지정했습니다. 이는 MEDIUMTEXT의 최대 길이보다 1 큰 숫자입니다. length 는 int 타입이라 LONGTEXT 의 최대값인 4_294_967_295 를 지정할 수 없어서 이렇게 지정했습니다. 이 값이 최대 길이를 나타내는 것이 아니기 때문에, 혼란 스러울 수 있을 것 같습니다. 팀이 있다면 합의를 하는게 좋을 것 같습니다.

LONGTEXT 가 가변길이라 문제가 없을 것 같기는 한데, 그래도 최대값이 너무 크니 찝찝해서 좀 블로그 글들을 찾아본 결과 TEXT 를 사용할 때 주의사항 외에는 특별한 점은 보이지 않습니다. 물론 진짜 4GB 정도 되는 데이터가 저장할게 아니라면 그에 따른 조치는 필요하겠죠. 애플리케이션 단에서 용량을 제한한다던지... 그런데 MEDIUMTEXT(16MB) 와 LONGTEXT(4GB)는 갭이 너무 큰 것 같습니다.

## User 클래스 수정하기

Post에만 관계를 추가하는 단방향 관계를 만들 수도 있다고 합니다. 하지만 원래 관계 데이터 모델에서는 한 쪽에 외래키 추가하는 순간 양방향 관계가 되기도 하고, 현업이신 분이 정리한 글([링크](https://ym1085.github.io/jpa/JPA-%EC%9D%BC%EB%8C%80%EB%8B%A4/))을 보면 단방향 관계에서 문제점들을 지적하고 있어, User 클래스는 `@OneToMany` 를 추가합니다.

```java
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
@Entity
@Table(name = "users")
@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})
public class User implements UserDetails {

    // 기존 필드 생략

    @JsonIgnore
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    List<Post> posts;

    // UserDetails 상속받은 메서드 생략
}
```

### @JsonIgnoreProperties, @JsonIgnore

`@JsonIgnoreProperties` 와 `@JsonIgnore`는 책에서 배운게 있어서 추가했습니다. User를 참조하고 다시 Post로 가서 User 를 참조하면 무한 루프가 걸려버립니다. 라고 책에서 배웠는데, 이게 당시 책에서 사용했던 Spring Data REST 때문인건지는 잘 모르겠어서, 뒤에가서 실험해 보도록 하겠습니다.

### @OneToMany

그리고 mappedBy 는 Post 에 있는 user 컬럼을 가리키고, cascade 는 ON DELETE CASCADE 처럼 부모인 User 의 id 가 갱신될 때 무엇을 할 것인지 지정합니다. 저는 ALL 로 해놔서 회원이 탈퇴하면 회원이 작성한 게시글 또한 모두 삭제됩니다.

애플리케이션의 기획에 따라 달라질 수 있습니다. 만약 삭제 되지 않게 하려면, REMOVE 만 제외하고 지정하는 등 이외에는 상황에 따라 골라서 설정하면 됩니다.

```java
@OneToMany(mappedBy = "user", cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
```

그리고 @Builder 를 추가했는데, Test 코드에서 new 연산자를 이용해 인스턴스를 만드는게 문제가 생겼기 때문입니다.

### MySQL 에서 자동 생성되는 CREATE 문

의도한 대로 잘 생성이 되는지 확인해보겠습니다.

```
Hibernate: 
    create table posts (
        created_at datetime(6),
        id bigint not null auto_increment,
        updated_at datetime(6),
        user bigint,
        title varchar(255) not null,
        content longtext not null,
        primary key (id)
    ) engine=InnoDB
Hibernate: 
    create table users (
        created_at datetime(6),
        id bigint not null auto_increment,
        email varchar(255) not null,
        password varchar(255) not null,
        primary key (id)
    ) engine=InnoDB
Hibernate: 
    alter table users 
       add constraint UK_6dotkott2kjsp8vw4d0m25fb7 unique (email)
Hibernate: 
    alter table posts 
       add constraint FKpmwxerjkloqfpmwognesh3xsn 
       foreign key (user) 
       references users (id)
```

posts 테이블의 content 가 LONGTEXT로 이상 없이 설정이 됐고, 외래키 또한 users 테이블의 id에 관계 설정이 잘 된 것을 볼 수 있습니다.

## User 테스트 코드 수정

아무 이유없이 객체에 @Builder를 User 클래스에 추가하기 싫어서 미루고 있었는데, 미루다가 더 귀찮아졌습니다.

@AllArgConstructor 를 선언해놓아서 아래와 같이 User 인스턴스를 만들고 있었습니다.

```java
User user = new User(
        null,
        "limvik@limvik.com",
        passwordEncoder.encode("password"),
        null);
```

이제 Builder 를 이용해서 수정해주면서, 모든 테스트에서 필요하므로 중복 코드 제거할겸 @BeforeEach로 테스트 메서드마다 다시 생성하도록 추가해줍니다.

```java
@BeforeEach
void init() {
    user = User.builder()
            .email("limvik@limvik.com")
            .password(passwordEncoder.encode("password"))
            .build();
}
```

변경이 없으니 @BeforeAll 이 더 맞는거 같기는 한데, static 변수 만들기 싫어서 @BeforeEach 했습니다.

## 게시글(Post) JSON 테스트 추가

### Serialization 테스트

먼저 serialization 테스트를 수행하기 위해 post.json 파일을 추가합니다. 실제로는 비밀번호가 오가는 경우는 없겠지만, 아직 테스트 코드 작성 능력의 한계로 User 클래스가 가진 필드를 그대로 JSON 프로퍼티에 작성합니다.

```json
{
  "id": 1,
  "title": "title of limvik",
  "content": "content of limvik",
  "createdAt": "2023-08-01T00:00:00",
  "updatedAt": "2023-08-01T00:00:00",
  "user": {
    "id": 1,
    "email": "limvik@limvik.com",
    "password": "password",
    "createdAt": "2023-08-01T00:00:00"
  }
}
```

다음은 serialization 테스트를 작성한 전체 코드입니다.

```java
package com.limvik.wantedpreonboardingbackend;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import com.limvik.wantedpreonboardingbackend.domain.User;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.json.JsonTest;
import org.springframework.boot.test.json.JacksonTester;

import java.io.IOException;
import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.assertThat;

@JsonTest
public class PostJsonTests {
    @Autowired
    JacksonTester<Post> json;

    private Post post;

    @BeforeEach
    void init() {
        User user = User.builder()
                .id(1L)
                .email("limvik@limvik.com")
                .password("password")
                .createdAt(getLocalDateTimeDummy())
                .build();

        post = Post.builder()
                .id(1L)
                .title("title of limvik")
                .content("content of limvik")
                .createdAt(getLocalDateTimeDummy())
                .updatedAt(getLocalDateTimeDummy())
                .user(user)
                .build();
    }

    private LocalDateTime getLocalDateTimeDummy() {
        return LocalDateTime.of(2023, 8, 1, 0, 0, 0);
    }

    @Test
    void PostSerializationTest() throws IOException {
        assertThat(json.write(post)).isStrictlyEqualToJson("post.json");
        assertThat(json.write(post)).hasJsonPathNumberValue("@.id");
        assertThat(json.write(post)).extractingJsonPathNumberValue("@.id")
                .isEqualTo(1);
        assertThat(json.write(post)).hasJsonPathStringValue("@.title");
        assertThat(json.write(post)).extractingJsonPathStringValue("@.title")
                .isEqualTo("title of limvik");
        assertThat(json.write(post)).hasJsonPathStringValue("@.content");
        assertThat(json.write(post)).extractingJsonPathStringValue("@.content")
                .isEqualTo("content of limvik");
        assertThat(json.write(post)).hasJsonPath("@.createdAt");
        assertThat(json.write(post)).extractingJsonPathStringValue("@.createdAt")
                .isEqualTo("2023-08-01T00:00:00");
        assertThat(json.write(post)).hasJsonPath("@.updatedAt");
        assertThat(json.write(post)).extractingJsonPathStringValue("@.updatedAt")
                .isEqualTo("2023-08-01T00:00:00");
        assertThat(json.write(post)).hasJsonPath("@.user");
        assertThat(json.write(post)).extractingJsonPathValue("@.user")
                .extracting("id").isEqualTo(1);
    }
    
}

```

PASSED! 그런데, 클래스를 먼저 만들어버리니 TDD 사이클이 아니게되네요. 데이터베이스 설계를 미리 해놔서 약간 애매합니다. ERD 보면서 테스트를 먼저 만들었어야 했나 싶기도...?

```
PostJsonTests > PostSerializationTest() PASSED
```

다음은 반대 방향으로도 테스트를 수행해야겠습니다.

### Deserialization 테스트

```java
@Test
void PostDeserializationTest() throws IOException {
    String expected = """
            {
              "id": 1,
              "title": "title of limvik",
              "content": "content of limvik",
              "createdAt": "2023-08-01T00:00:00",
              "updatedAt": "2023-08-01T00:00:00",
              "user": {
                "id": 1,
                "email": "limvik@limvik.com",
                "password": "password",
                "createdAt": "2023-08-01T00:00:00"
              }
            }
            """;

    assertThat(json.parse(expected))
            .isEqualTo(post);

    assertThat(json.parseObject(expected).getId()).isEqualTo(1L);
    assertThat(json.parseObject(expected).getTitle()).isEqualTo("title of limvik");
    assertThat(json.parseObject(expected).getContent()).isEqualTo("content of limvik");
    assertThat(json.parseObject(expected).getCreatedAt()).isEqualTo(getLocalDateTimeDummy());
    assertThat(json.parseObject(expected).getUpdatedAt()).isEqualTo(getLocalDateTimeDummy());
    assertThat(json.parseObject(expected).getUser()).isInstanceOf(User.class);
    assertThat(json.parseObject(expected).getUser().getId()).isEqualTo(1L);
}
```

Deserialization 테스트도 이상 없이 통과합니다.

```
PostJsonTests > PostDeserializationTest() PASSED
```

다음은 구현하기 전에 '새로운 게시글 생성 엔드포인트'에 관한 내용을 정의합니다.

## 새로운 게시글 생성 엔드포인트

### 경로 및 자원명

사용자(user)의 경로를 지정할 때는 단일 자원에 대한 요청밖에 없다고 생각해서 단수로 정했는데(관리자가 사용자 목록을 불러오는 경우를 당시에는 고려 안했던게 갑자기 떠오릅니다.), 게시글(post)은 목록을 불러올 때 여러 자원에 대한 정보를 불러와야 하므로 자원명을 복수형으로 정하고 경로를 아래와 같이 지정합니다.

> /api/v1/posts

### HTTP method

`생성`하는 것이니 큰 고민이 필요 없습니다. `POST` 를 사용합니다.

`PUT`을 생성에도 사용하기는 하지만, 현재 배운대로면 이미 자원의 위치를 알고 있을 때 `PUT`을 사용합니다. 하지만, 게시글의 식별자인 id 는 데이터베이스에서 자동 생성되므로 클라이언트에서 알 수 없습니다.

### HTTP Response Status Code

게시글을 정상적으로 생성한 경우, 제목이나 내용이 비어있는 경우, 권한 없는 사용자가 요청한 경우로 나눠볼 수 있겠습니다.

정상적으로 생성한 경우 201(Created)로 쉽게 선택할 수 있습니다. 제목이나 내용이 비어있는 경우는 유효성 검사에서 실패한 것으로, 앞서 유효성 검사 실패한 것과 같이 400(Bad Request) 로 지정합니다. 마지막으로 게시글 생성 권한이 없는 경우(토큰 기간 만료 등 유효하지 않은 토큰을 보낸 경우도 포함)는 401(Unauthorized)로 지정합니다.

### 요청 양식

이전에 클라이언트는 로그인을 통해서 토큰을 받아갔으므로, 로그인 했다면 유효한 JWT(JSON Web Token)를 서버로 보내야 합니다. 

JWT를 JSON 에 포함시킬 수도 있지만, 헤더에 추가해서 보내는 것이 더 일반적인 것으로 판단되어 헤더에 포함시키려고 합니다.

자주 보이는 방식은 `X-AUTH-TOKEN` 헤더를 사용하는 방식과 `Authorization` 헤더에 `bearer` prefix 를 사용하는 방식이 있습니다.

Authorization 헤더에 bearer prefix를 사용하는 방식이 원래 있었는데, OAuth 2.0에서 갖다 쓴건지 OAuth 2.0 을 만들면서 사용하게 된건지 궁금했는데, swagger 문서([링크](https://swagger.io/docs/specification/authentication/bearer-authentication/))에 의하면 후자라고 합니다. RFC 문서([링크](https://datatracker.ietf.org/doc/html/rfc6750))도 bearer 와 관련해서 나온게 OAuth 2.0에서 어떻게 사용하는지에 대한 것 뿐입니다.

`X-` 가 붙었다면 사람들이 임의로 사용하는 헤더이기는 하겠지만, OAuth 2.0 이 아닐 때 Token 을 포함시키기에 적합한 헤더라고 생각됩니다.

인프런에 관련 질문([링크](https://www.inflearn.com/questions/178947/auth-js-gt-line-6-quot-x-auth-token-quot-%EA%B4%80%EB%A0%A8-%EC%A7%88%EB%AC%B8))에 대한 답변이 있기는 한데, 출처가 없어서 아쉽습니다.

그럼 `X-AUTH-TOKEN` 에 로그인을 통해서 받은 JWT를 포함시키는 것으로 하고, HTTP request body 에는 JSON 으로 게시글 제목과 내용을 포함시켜 요청하도록 합니다.

### 응답 양식

응답으로 body에는 특별히 넣을게 없지만, 강의에서 배운대로 Location 헤더에 새로이 생성한 게시글의 경로를 추가하여 반환합니다.

### 새로운 게시글 생성 엔드포인트 정리

- Request

> POST /api/v1/posts
>
> X-AUTH-TOKEN: {JWT}

```json
{
  "title": "title",
  "contents": "contents"
}
```

- Response
	- 성공 시 Location 헤더에 게시글 경로 추가
	- /api/v1/posts/{id}

- Response Status Code
	- 게시글 생성 성공: 201(Created)
	- 제목이나 내용이 비어있는 경우: 400(Bad Request)
	- 게시글 생성 권한 없는 경우: 401(Unauthorized)

## 새로운 게시글 생성 요청 테스트

### User, Post 클래스 수정

이번에는 요청하는 테스트를 하려고 보니, 테스트 데이터베이스인 h2 가 테이블을 생성을 합니다. 그런데 컬럼이름도 user가 있으니 오류가 발생합니다.

h2에서 예약어는 어디든 안되나봅니다. 그래서 다음과 같이 수정해줍니다.

```java
// Post.java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "userId")
private User userId;

// User.java
@JsonIgnore
@OneToMany(mappedBy = "userId", cascade = CascadeType.ALL)
List<Post> posts;
```

h2 덕분에 컬럼명이 더 명확해졌습니다.

```
Hibernate: 
    create table posts (
        created_at datetime(6),
        id bigint not null auto_increment,
        updated_at datetime(6),
        user_id bigint,
        title varchar(255) not null,
        content longtext not null,
        primary key (id)
    ) engine=InnoDB
Hibernate: 
    create table users (
        created_at datetime(6),
        id bigint not null auto_increment,
        email varchar(255) not null,
        password varchar(255) not null,
        primary key (id)
    ) engine=InnoDB
Hibernate: 
    alter table users 
       add constraint UK_6dotkott2kjsp8vw4d0m25fb7 unique (email)
Hibernate: 
    alter table posts 
       add constraint FK5lidm6cqbc7u4xhqpxm898qme 
       foreign key (user_id) 
       references users (id)
```

### PostRequestTests

테스트 독립성과 테스트 시간 사이에서 고민을 좀 했습니다.

테스트 시간을 빠르게 하기 위해서, @TestInstance(TestInstance.Lifecycle.PER_CLASS) 로 클래스 단위로 테스트를 수행하게 하고, @BeforeAll 이 표시된 메서드에서 사용자를 한 번만 저장하게 하려고 했습니다.

그런데 아직 테스트의 갯수가 적어서 시간 차이가 얼마 없다보니, 테스트를 독립적으로 수행해야 한다는 원칙에 맞춰 @DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD) 를 사용하여 테스트 마다 컨텍스트를 생성하고, @BeforeEach 를 표시한 메서드에서 사용자 정보를 저장하게 하였습니다.

이는 토큰(JWT)에 있는 정보를 바탕으로 데이터베이스에서 사용자를 조회할 때 ID를 이용하기 위함입니다.

```java
package com.limvik.wantedpreonboardingbackend;

import com.limvik.wantedpreonboardingbackend.config.JwtConfig;
import com.limvik.wantedpreonboardingbackend.domain.Post;
import com.limvik.wantedpreonboardingbackend.domain.User;
import com.limvik.wantedpreonboardingbackend.jwt.JwtProvider;
import com.limvik.wantedpreonboardingbackend.repository.UserRepository;
import io.jsonwebtoken.Header;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.*;
import org.springframework.test.annotation.DirtiesContext;

import java.nio.charset.StandardCharsets;
import java.util.Date;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public class PostRequestTests {
    @Autowired
    TestRestTemplate restTemplate;

    @Autowired
    JwtProvider jwtProvider;

    @Autowired
    JwtConfig jwtConfig;

    @Autowired
    UserRepository userRepository;

    @BeforeEach
    void init() {
        userRepository.save(User.builder()
                .email("limvik@limvik.com")
                .password("password")
                .build());
    }

    @Test
    @DisplayName("유효한 토큰으로 게시글 생성 요청")
    void shouldCreateAPostIfValidToken() {

        Post newPost = Post.builder()
                .title("title")
                .content("content")
                .build();

        HttpHeaders headers = new HttpHeaders();
        String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
        headers.set("X-AUTH-TOKEN", jwt);
        HttpEntity<Post> request = new HttpEntity<>(newPost, headers);

        ResponseEntity<Void> createResponse = restTemplate
                .exchange("/api/v1/posts", HttpMethod.POST, request, Void.class);

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getHeaders().getLocation()).isEqualTo("/api/v1/posts/1");

    }

    @Test
    @DisplayName("유효한 토큰으로 `제목`이 비어있는 게시물 생성 요청")
    void shouldNotCreateAPostIfValidTokenAndNullTitle() {
        Post newPost = Post.builder()
                .title("")
                .content("content")
                .build();

        HttpHeaders headers = new HttpHeaders();
        String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
        headers.set("X-AUTH-TOKEN", jwt);
        HttpEntity<Post> request = new HttpEntity<>(newPost, headers);

        ResponseEntity<Void> createResponse = restTemplate
                .exchange("/api/v1/posts", HttpMethod.POST, request, Void.class);

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
    }

    @Test
    @DisplayName("유효한 토큰으로 `내용`이 비어있는 게시물 생성 요청")
    void shouldNotCreateAPostIfValidTokenAndNullContent() {
        Post newPost = Post.builder()
                .title("title")
                .content("")
                .build();

        HttpHeaders headers = new HttpHeaders();
        String jwt = jwtProvider.generateToken(User.builder().id(1L).email("limvik@limvik.com").build());
        headers.set("X-AUTH-TOKEN", jwt);
        HttpEntity<Post> request = new HttpEntity<>(newPost, headers);

        ResponseEntity<Void> createResponse = restTemplate
                .exchange("/api/v1/posts", HttpMethod.POST, request, Void.class);

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
    }

    @Test
    @DisplayName("만료된 토큰으로 게시물 생성 요청")
    void shouldNotCreateAPostIfInvalidToken() {
        Post newPost = Post.builder()
                .title("title")
                .content("content")
                .build();

        HttpHeaders headers = new HttpHeaders();
        String jwt = generateInvalidToken();
        headers.set("X-AUTH-TOKEN", jwt);
        HttpEntity<Post> request = new HttpEntity<>(newPost, headers);

        ResponseEntity<Void> createResponse = restTemplate
                .exchange("/api/v1/posts", HttpMethod.POST, request, Void.class);

        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);

    }

    String generateInvalidToken() {
        Date now = new Date(new Date().getTime() - JwtConfig.JWT_EXPIRATION_MS);

        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
                .setIssuer(jwtConfig.getJwtIssuer())
                .setIssuedAt(now)
                .setExpiration(now)
                .setSubject("1")
                .claim("email", "limvik@limvik.com")
                .signWith(Keys.hmacShaKeyFor(jwtConfig.getJwtKey().getBytes(StandardCharsets.UTF_8)))
                .compact();
    }

}
```

테스트에서 header 를 추가하는 방법도 처음 해봤습니다. 처음에 몰라가지고 request를 만든 후에 header를 추가했었는데, 안됩니다.

그리고 테스트를 수행해보면, 404(Not Found)가 나옵니다. 경로가 아예 존재하지 않으면 그냥 404가 나옵니다.

```
expected: 400 BAD_REQUEST
 but was: 404 NOT_FOUND
```

## 엔드포인트 구현하기

### PostController

이제 경로를 추가하기 위해서 먼저 Controller를 추가합니다.

```java
package com.limvik.wantedpreonboardingbackend.controller;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import com.limvik.wantedpreonboardingbackend.domain.User;
import com.limvik.wantedpreonboardingbackend.service.PostService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;

@RequiredArgsConstructor
@RestController
@RequestMapping("/api/v1/posts")
public class PostController {

    private final PostService postService;

    @PostMapping
    public ResponseEntity<Void> createNewPost(@AuthenticationPrincipal User user,
                                              @Valid @RequestBody Post newPost,
                                              UriComponentsBuilder ucb) {
        Post savedNewPost = postService.create(newPost, user);
        return ResponseEntity.created(buildLocationOfNewPostUri(savedNewPost, ucb)).build();
    }

    private URI buildLocationOfNewPostUri(Post savedPost,
                                          UriComponentsBuilder ucb) {
        return ucb
                .path("/api/v1/posts/{id}")
                .buildAndExpand(savedPost.getId())
                .toUri();
    }

}
```

새로운 게시글을 저장하고, Location 헤더에 URI를 추가합니다. User 를 저장할 때와 로직이 동일합니다.

Service가 필요하니 Service 를 추가합니다.

### PostService

게시글의 작성자를 식별하기 위해 사용자 id를 설정한 후에 저장합니다.

```java
package com.limvik.wantedpreonboardingbackend.service;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import com.limvik.wantedpreonboardingbackend.domain.User;
import com.limvik.wantedpreonboardingbackend.repository.PostRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@RequiredArgsConstructor
@Service
public class PostService {

    private final PostRepository postRepository;

    public Post create(Post newPost, User user) {
        newPost.setUserId(user);
        return postRepository.save(newPost);
    }
}
```

다음으로 필요한 Repository를 구현합니다.

### PostRepository

save 메서드는 자동으로 구현되므로, JpaRepository 만 상속받으면 됩니다.

```java
package com.limvik.wantedpreonboardingbackend.repository;

import com.limvik.wantedpreonboardingbackend.domain.Post;
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long> {

}
```

## 테스트 중간 점검

토큰이 유효한지는 아직 검증하는 로직이 없으니, 유효한 토큰으로 새로운 게시물이 생성되는 테스트는 통과하고 나머지는 기대하는 Response Status Code와 다르므로 테스트에 실패해야합니다.

그리고 테스트를 해보면 아래와 같습니다.

```
expected: "/api/v1/posts/1"
 but was: http://localhost:3598/api/v1/posts/1
```

나머지는 동일하게 201(Created)를 받아서 테스트에 실패합니다.

```
expected: 400 BAD_REQUEST
 but was: 201 CREATED

expected: 401 UNAUTHORIZED
 but was: 201 CREATED
```

호스트가 붙는걸 생각을 못해서, 테스트를 수정해주겠습니다.

### 테스트 수정

host 상관없이 경로를 확인할 수 있는 `hasPath` 메서드를 사용해서 수정하였습니다.

```java
assertThat(createResponse.getHeaders().getLocation()).hasPath("/api/v1/posts/1");
```

그러면 유효한 토큰으로 게시글 생성 요청하는 테스트는 PASSED 가 되는 것을 볼 수 있습니다.

```
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
```

다음은 JWT 검증하기 위해서 Spring Security 관련해서 구현하고 설정을 해줘야겠습니다.

## Spring Security

큰 흐름은 로그인할 때와 비슷합니다. 흐름을 다시 생각해보자면 Filter 에서 Http 요청을 가로채고, 인증용 Token을 만들어서 AuthenticationManager로 넘깁니다. AuthenticationManager의 구현체인 ProviderManager 에 설정한 Provider로 인증을 완료합니다. 그리고 인증 여부에 따른 작업을 수행합니다.

Spring Security 에 JWT 처리하는게 없나 싶어서 살펴봤는데, OAuth2 Server 용 라이브러리를 추가해야 JWT 관련한 클래스를 사용할 수 있습니다.

그러면... 복사해서 적당히 수정을 하는 방향으로 가보겠습니다.

### OAuth2 Server 라이브러리 살펴보기

OAuth2 Server 의 패키지와 클래스 목록([링크](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/server/resource/authentication/package-summary.html))을 살펴보면서 무엇을 가져다 쓸까 고민을 시작합니다.

먼저 Filter는 [BearerTokenAuthenticationFilter](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/server/resource/web/authentication/BearerTokenAuthenticationFilter.html)를 벤치마킹(?) 하기로 합니다. 

### JwtAuthenticationFilter 

BearerTokenAuthenticationFilter([Github 링크](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-resource-server/src/main/java/org/springframework/security/oauth2/server/resource/web/authentication/BearerTokenAuthenticationFilter.java))는 @author Jeongjin Kim 한국인 이름이 보이니 괜히 반가운 마음이 듭니다.

뻘 소리였구요. 음... Filter는 특정한 요청을 필터링하고 작업을 넘겨주는게 주 업무라 기존에 보던 것과 크게 달라 보이는 것은 없습니다.

게시글 목록은 아무나 요청할 수 있으므로, 이전에 EmailPasswordAuthenticationFilter를 만들었을 때와는 달리, `X-AUTH-TOKEN` 헤더에 토큰이 없는 경우는 다음 Filter 로 넘어가도록 해야합니다. 그 이외에는 EmailPasswordAuthenticationFilter를 만들었을 때와 마찬가지로 인증을 진행합니다.

```java
package com.limvik.wantedpreonboardingbackend.securityfilter;

import io.jsonwebtoken.MalformedJwtException;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.context.SecurityContextHolderStrategy;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.NoSuchElementException;

public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder
            .getContextHolderStrategy();

    private final AuthenticationManager authenticationManager;

    public JwtAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        JwtAuthenticationToken token;
        try {
            token = new JwtAuthenticationConverter().convert(request);
        } catch (MalformedJwtException e) {
            // JWT 형식이 아닌 경우
            buildFailedResponse(response, e);
            return;
        } catch (NoSuchElementException e) {
            // JWT 가 필요 없는 접근인 경우
            filterChain.doFilter(request, response);
            return;
        }

        try {
            securityContextHolderStrategy.getContext().setAuthentication(authenticationManager.authenticate(token));
            filterChain.doFilter(request, response);
        } catch (AuthenticationException e) {
            buildFailedResponse(response, e);
        }

    }

    private void buildFailedResponse(HttpServletResponse response, Exception e) throws IOException {
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        String jsonResponse = "{\"error\": \"%s\"}".formatted(e.getMessage());
        response.getWriter().write(jsonResponse);
    }

}
```

EmailPasswordAuthenticationFilter 를 만들 때는 UsernamePasswordAuthenticationToken을 적당히 가져다 썼지만, 이번에는 JwtAuthenticationToken 을 만들어주어야 하고, 저번처럼 JwtAuthenticationConverter 도 만들어줘야 합니다.

먼저 Converter를 만들겠습니다.

### JwtAuthenticationConverter

이번에는 header에 있는 데이터만 가져오면 돼서 Converter가 간단합니다.

```java
package com.limvik.wantedpreonboardingbackend.converter;

import com.limvik.wantedpreonboardingbackend.jwt.JwtAuthenticationToken;
import com.limvik.wantedpreonboardingbackend.jwt.JwtProvider;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.security.SignatureException;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.security.web.authentication.AuthenticationConverter;

import java.util.NoSuchElementException;
import java.util.Optional;

public class JwtAuthenticationConverter implements AuthenticationConverter {
    
    private final JwtProvider jwtProvider;
    
    public JwtAuthenticationConverter(JwtProvider jwtProvider) {
        this.jwtProvider = jwtProvider;
    }

    @Override
    public JwtAuthenticationToken convert(HttpServletRequest request)
            throws MalformedJwtException, ExpiredJwtException, SignatureException, NoSuchElementException {
        
        String jwt = Optional.of(request.getHeader("X-AUTH-TOKEN")).orElseThrow();
        return new JwtAuthenticationToken(jwtProvider.parse(jwt));

    }
}

```

이를 위해서 JwtProvider 에 parse 메서드를 추가했습니다.

```java
public Jws<Claims> parse(String token) {
    return Jwts.parserBuilder()
            .requireIssuer(jwtConfig.getJwtIssuer())
            .setSigningKey(Keys.hmacShaKeyFor(jwtConfig.getJwtKey().getBytes(StandardCharsets.UTF_8)))
            .build()
            .parseClaimsJws(token);
}
```

그리고 Filter 에서도 Jwt 와 관련해서 예외가 발생했을 때는 모두 똑같이 다음 Filter 없이 과정을 끝내야 하므로 MalformedJwtException 에서 JwtException 으로 통일했습니다.

특히 다른 예제를 보고서 만료된 토큰을 직접 확인해야할 줄 알았는데, jjwt 라이브러리의 DefaultJwtParser 가 parsing 할 때 만료 여부를 체크하고 ExpiredJwtException 을 던지고 있어서([Github 링크](https://github.com/jwtk/jjwt/blob/master/impl/src/main/java/io/jsonwebtoken/impl/DefaultJwtParser.java#L530)) 간단하게 처리할 수 있게 됐습니다. 실제로 되는지는 테스트를 해봐야겠습니다.

```java
// JwtAuthenticationFilter.java 수정

JwtAuthenticationToken token;
try {
    token = new JwtAuthenticationConverter(jwtProvider).convert(request);
} catch (JwtException e) {
    buildFailedResponse(response, e);
    return;
} catch (NoSuchElementException e) {
    // JWT 가 필요 없는 접근인 경우
    filterChain.doFilter(request, response);
    return;
}
```

다음은 JwtAuthenticationToken 을 만들어야 겠습니다.

### JwtAuthenticationToken 

OAuth2 Server 라이브러리의 JwtAuthenticationToken([Github 링크](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-resource-server/src/main/java/org/springframework/security/oauth2/server/resource/authentication/JwtAuthenticationToken.java))을 따라서 AbstractAuthenticationToken([Github 링크](https://github.com/spring-projects/spring-security/blob/main/core/src/main/java/org/springframework/security/authentication/AbstractAuthenticationToken.java))을 상속받습니다.

실제로는 JwtAuthenticationToken 은 AbstractAuthenticationToken 추상 클래스를 상속받은 AbstractOAuth2TokenAuthenticationToken\<T extends OAuth2Token>([Github 링크](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-resource-server/src/main/java/org/springframework/security/oauth2/server/resource/authentication/AbstractOAuth2TokenAuthenticationToken.java))을 상속받고 있지만, OAuth2 에서 사용하는 Token이 여러 종류이기 때문으로 보입니다.

그래서 저는 바로 AbstractAuthenticationToken 추상 클래스를 상속받습니다.

```java
package com.limvik.wantedpreonboardingbackend.jwt;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.Transient;

import java.util.Collection;

@Transient
public class JwtAuthenticationToken extends AbstractAuthenticationToken {

    private final Jws<Claims> token;

    private Object credentials;

    private Object principal;

    public JwtAuthenticationToken(Jws<Claims> token) {
        super(null);
        this.token = token;
    }

    public JwtAuthenticationToken(Jws<Claims> token, Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.setAuthenticated(true);
        this.token = token;
    }

    @Override
    public Object getCredentials() {
        return credentials;
    }

    @Override
    public Object getPrincipal() {
        return principal;
    }

    public Claims getClaims() {
        return token.getBody();
    }

}
```

token에 권한(authorities)을 그냥 setter로 설정하면 되는게 아닌가 싶지만, 미리 만들어두신 분들이 괜히 생성자로만 권한 설정이 가능하게 해놓은 것은 아니겠다는 생각이 들어서, 새로운 인스턴스를 생성할 때만 권한 설정이 가능하게 해두었습니다.

그리고 getTokenAttributes 메서드를 따라서, `getClaims` 메서드를 만들었습니다. 당장의 필요성이 떠오르지 않아 지우고 싶긴 하지만, 일단은 어느정도 미리 만들어진 것을 따라가보면서 필요 없으면 지우기로 했습니다.

다음으로 token에 credentials 를 저장하지는 않으니까 필요없는데, 부모인 Authentication 인터페이스에 있는 메서드(getCredentials)라 없앨 수는 없었습니다.

OAuth2 Server 라이브러리의 AbstractOAuth2TokenAuthenticationToken 에서는 credentials에 null을 허용해서 해결하고 있습니다.

```java
protected AbstractOAuth2TokenAuthenticationToken(T token, Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {

    super(authorities);
    Assert.notNull(token, "token cannot be null");
    Assert.notNull(principal, "principal cannot be null");
    this.principal = principal;
    this.credentials = credentials;
    this.token = token;
}
```

이제 인증을 수행할 AuthenticationProvider 를 상속받아 JwtAuthenticationProvider를 만들어야겠습니다.

### JwtAuthenticationProvider에 대한 고찰...

OAuth2 Server 라이브러리에 있는 JwtAuthenticationProvider([Github 링크](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-resource-server/src/main/java/org/springframework/security/oauth2/server/resource/authentication/JwtAuthenticationProvider.java))를 보면, 여기서 Bearer 토큰을 jwt로 변환합니다. 

```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    BearerTokenAuthenticationToken bearer = (BearerTokenAuthenticationToken) authentication;
    Jwt jwt = getJwt(bearer);
    AbstractAuthenticationToken token = this.jwtAuthenticationConverter.convert(jwt);
    if (token.getDetails() == null) {
        token.setDetails(bearer.getDetails());
    }
    this.logger.debug("Authenticated token");
    return token;
}
```

저는 Filter에서 했었는데, 좀 다릅니다. 앞에서는 언급 안했지만 사실 처음에는 token에 있는 정보를 가지고 Provider 에서 데이터베이스를 조회해서 존재하는 사용자 정보인지 확인하려고 했습니다.

그런데 토큰에 문제가 있다면 예외가 던져질 것이고, 유효한 토큰이지만 사용자가 탈퇴한 경우에는 필수적 관계에 있는 사용자 정보가 없어 게시글이 작성되지 않을테니 굳이 데이터베이스를 조회할 필요가 없겠다는 생각이 들었습니다.

그러면...! Provider에서 할 일이 token에 있는 사용자 정보를 추출해서 저장하는 것 말고는 할 일이 없어집니다. OAuth2 Server 라이브러리에서는 BearerAuthenticationToken과 JwtAuthenticationToken이 나누어 작업을 해서 그렇습니다. 만약 제가 라이브러리를 따라한다면 XAuthAuthenticationToken을 만들어야겠네요.

BearerAuthenticationToken([Github 링크](https://github.com/spring-projects/spring-security/blob/main/oauth2/oauth2-resource-server/src/main/java/org/springframework/security/oauth2/server/resource/authentication/BearerTokenAuthenticationToken.java))을 보면 문자열 상태인 Bearer Token 을 담아두는 컨테이너입니다. AuthenticationManager로 넘겨주기위한 래퍼 정도라 볼 수 있겠습니다. API 문서([링크](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/server/resource/authentication/BearerTokenAuthenticationToken.html))의 설명은 아래와 같습니다.

> An Authentication that contains a Bearer Token. Used by BearerTokenAuthenticationFilter to prepare an authentication attempt and supported by JwtAuthenticationProvider.

따라가고 말고를 떠나서, Filter에서 Jwt가 유효한지 여부에 대해 인증을 수행했습니다.

좀 귀찮기는 하지만 Filter를 XAuthAuthenticationToken 을 만들고 JwtAuthenticationToken은 JwtAuthenticationProvider에서 생성하는 것으로 수정하겠습니다.

### XAuthAuthenticationToken

앞서 언급한대로 문자열인 jwt를 보관하는 컨테이너 정도로 만듭니다.

```java
package com.limvik.wantedpreonboardingbackend.jwt;

import org.springframework.security.authentication.AbstractAuthenticationToken;

public class XAuthAuthenticationToken extends AbstractAuthenticationToken  {
    private final String token;

    public XAuthAuthenticationToken(String token) {
        super(null);
        this.token = token;
    }

    public String getToken() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }
}
```

### XAuthAuthenticationConverter

JwtAuthenticationConverter는 XAuth용으로 변경합니다.

```java
package com.limvik.wantedpreonboardingbackend.converter;

import com.limvik.wantedpreonboardingbackend.jwt.XAuthAuthenticationToken;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.security.web.authentication.AuthenticationConverter;

import java.util.NoSuchElementException;
import java.util.Optional;

public class XAuthAuthenticationConverter implements AuthenticationConverter {

    @Override
    public XAuthAuthenticationToken convert(HttpServletRequest request)
            throws NoSuchElementException {
        String jwt = Optional.of(request.getHeader("X-AUTH-TOKEN")).orElseThrow();
        return new XAuthAuthenticationToken(jwt);

    }
}
```

### XAuthAuthenticationFilter

X-AUTH-TOKEN 헤더가 있는 request를 Filtering 하는 Filter 이니 Filter 이름도 바꿔주고 jwt 문자열을 parsing 하면서 발생하는 예외도 없으므로, 관련 예외처리를 제거합니다.

너무 긴 관계로 주요 변경 부분만 첨부합니다.

```java
XAuthAuthenticationToken token;
try {
    token = new XAuthAuthenticationConverter().convert(request);
} catch (NoSuchElementException e) {
    // JWT 가 필요 없는 접근인 경우
    filterChain.doFilter(request, response);
    return;
}
```

JwtAuthenticationConverter는 왜 없나 생각했었는데, convert 메서드는 request를 인수로 받으니 없을 수 밖에 없었습니다.

그럼 이제 다시 원래 하려고 했던, JwtAuthenticationProvider 구현으로 넘어갑니다.

### JwtAuthenticationProvider

이제 provider에서 유효한 토큰인지 여부를 판단합니다.

```java
package com.limvik.wantedpreonboardingbackend.jwt;

import com.limvik.wantedpreonboardingbackend.domain.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.JwtException;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.*;

public class JwtAuthenticationProvider implements AuthenticationProvider {

    private final JwtProvider jwtProvider;

    public JwtAuthenticationProvider(JwtProvider jwtProvider) {
        this.jwtProvider = jwtProvider;
    }

    @Override
    public Authentication authenticate(Authentication authentication)
            throws JwtException, AuthenticationException {

        var xAuthToken = (XAuthAuthenticationToken) authentication;
        var jws = jwtProvider.parse(xAuthToken.getToken());
        var jwtToken = new JwtAuthenticationToken(jws, List.of(new SimpleGrantedAuthority(getAuthorities(jws))));
        jwtToken.setDetails(getDetails(jwtToken));
        return jwtToken;
    }

    private String getAuthorities(Jws<Claims> jws) {
        return ((LinkedHashMap<String, String>) jws.getBody().get("roles", ArrayList.class).get(0))
                .get("authority");
    }

    private UserDetails getDetails(JwtAuthenticationToken jwtToken) {
        var claims = jwtToken.getClaims();
        return User.builder()
                .id(Long.parseLong(claims.getSubject()))
                .email(claims.get("email").toString()).build();
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return XAuthAuthenticationToken.class.isAssignableFrom(authentication);
    }
}

```

### SecurityConfig

이제 게시글(post)와 관련된 경로에 대한 SecurityFilterChain을 설정합니다.

/api/v1/posts 와 관련된 경로는 모두 이 SecurityFilterChain을 거치도록 securityMatcher 메서드를 이용해 설정합니다. 그리고 지금 만들고 있는 /api/v1/posts 에 대한 POST 요청은 USER만 가능하도록 추가합니다. 나중에 필요한 경로는 그때 추가할 계획입니다.

```java
@Bean
public SecurityFilterChain postFilterChain(HttpSecurity http) throws Exception{
    return http
            .securityMatcher("/api/v1/posts/**")
            .authorizeHttpRequests(auth -> {
                auth.requestMatchers(new AntPathRequestMatcher("/api/v1/posts", "POST")).hasRole("USER");})
            .csrf(AbstractHttpConfigurer::disable)
            .logout(AbstractHttpConfigurer::disable)
            .anonymous(AbstractHttpConfigurer::disable)
            .sessionManagement(config -> config.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterAfter(new XAuthAuthenticationFilter(getJwtProviderManager()),
                    RequestCacheAwareFilter.class)
            .build();
}

private AuthenticationManager getJwtProviderManager() {
    return new ProviderManager(new JwtAuthenticationProvider(jwtProvider));
}
```

### 테스트 수행해보기

```
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > 만료된 토큰으로 게시물 생성 요청 FAILED
    org.opentest4j.AssertionFailedError: 
    expected: 401 UNAUTHORIZED
     but was: 500 INTERNAL_SERVER_ERROR
PostRequestTests > 유효한 토큰으로 `제목`이 비어있는 게시물 생성 요청 FAILED
    org.opentest4j.AssertionFailedError: 
    expected: 400 BAD_REQUEST
     but was: 201 CREATED
PostRequestTests > 유효한 토큰으로 `내용`이 비어있는 게시물 생성 요청 FAILED
    org.opentest4j.AssertionFailedError: 
    expected: 400 BAD_REQUEST
     but was: 201 CREATED
```

BAD_REQUEST인데 CREATED 가 나온거는 아직 유효성 검사를 안해서 FAILED가 되는게 맞는데, 만료된 토큰으로 게시물 생성 요청하는 것은 FAILED가 나오면 안됩니다.

정확한 원인은 만료된 토큰으로 인한 Exception을 처리해주지 않았기 때문입니다.

```
io.jsonwebtoken.ExpiredJwtException: JWT expired at 2023-08-09T13:40:46Z. Current time: 2023-08-10T13:40:46Z, a difference of 86400238 milliseconds.  Allowed clock skew: 0 milliseconds.
```

### ExpiredJwtException 처리하기

인증 중에 발생하는 Exception을 처리하는 Filter 로 갑니다.

```java
try {
    securityContextHolderStrategy.getContext().setAuthentication(authenticationManager.authenticate(token));
    filterChain.doFilter(request, response);
} catch (AuthenticationException | JwtException e) {
    securityContextHolderStrategy.clearContext();
    buildFailedResponse(response, e);
}
```

catch 절에서 AuthenticationException 뿐만 아니라 JwtException이 처리 되도록 추가해줍니다.

### 다시 테스트

다시 테스트를 해보면 이상 없이 PASSED가 시현되는 것을 볼 수 있습니다.

```
PostRequestTests > 만료된 토큰으로 게시물 생성 요청 PASSED
```

### SecurityFilter 로그 살펴보기

성공한 경우 Filter 로그를 살펴보겠습니다.

```
TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=Or [Mvc [pattern='/api/v1/user']], Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@5c3f9618, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@deb0c0e, org.springframework.security.web.header.HeaderWriterFilter@3c7e7ffd, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@51dd74a0, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@78652c15, org.springframework.security.web.session.SessionManagementFilter@45d28ab7, org.springframework.security.web.access.ExceptionTranslationFilter@504f2bcd, org.springframework.security.web.access.intercept.AuthorizationFilter@66fd9613]] (1/3)
TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=Or [Mvc [pattern='/api/v1/user/login']], Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@14c06f50, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@6230a15a, org.springframework.security.web.context.SecurityContextHolderFilter@e316971, org.springframework.security.web.header.HeaderWriterFilter@2c3f47ba, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@808f65, com.limvik.wantedpreonboardingbackend.securityfilter.EmailPasswordAuthenticationFilter@7225f871, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@56540a58, org.springframework.security.web.session.SessionManagementFilter@ca9ffc0, org.springframework.security.web.access.ExceptionTranslationFilter@527b989a, org.springframework.security.web.access.intercept.AuthorizationFilter@10bf2185]] (2/3)
TRACE o.s.security.web.FilterChainProxy - Trying to match request against DefaultSecurityFilterChain [RequestMatcher=Or [Mvc [pattern='/api/v1/posts/**']], Filters=[org.springframework.security.web.session.DisableEncodeUrlFilter@7b88dd58, org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@50720344, org.springframework.security.web.context.SecurityContextHolderFilter@4a3363c9, org.springframework.security.web.header.HeaderWriterFilter@4bd29a01, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@63cf6497, com.limvik.wantedpreonboardingbackend.securityfilter.XAuthAuthenticationFilter@5dce3e82, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@3c12f87d, org.springframework.security.web.session.SessionManagementFilter@5a4a8a33, org.springframework.security.web.access.ExceptionTranslationFilter@57cfd353, org.springframework.security.web.access.intercept.AuthorizationFilter@637bf67c]] (3/3)
DEBUG o.s.security.web.FilterChainProxy - Securing POST /api/v1/posts
TRACE o.s.security.web.FilterChainProxy - Invoking DisableEncodeUrlFilter (1/10)
TRACE o.s.security.web.FilterChainProxy - Invoking WebAsyncManagerIntegrationFilter (2/10)
TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderFilter (3/10)
TRACE o.s.security.web.FilterChainProxy - Invoking HeaderWriterFilter (4/10)
TRACE o.s.security.web.FilterChainProxy - Invoking RequestCacheAwareFilter (5/10)
TRACE o.s.security.web.FilterChainProxy - Invoking XAuthAuthenticationFilter (6/10)
TRACE o.s.s.w.c.SupplierDeferredSecurityContext - Created SecurityContextImpl [Null authentication]
TRACE o.s.s.authentication.ProviderManager - Authenticating request with JwtAuthenticationProvider (1/1)
TRACE o.s.security.web.FilterChainProxy - Invoking SecurityContextHolderAwareRequestFilter (7/10)
TRACE o.s.security.web.FilterChainProxy - Invoking SessionManagementFilter (8/10)
TRACE o.s.s.w.a.s.CompositeSessionAuthenticationStrategy - Preparing session with ChangeSessionIdAuthenticationStrategy (1/1)
TRACE o.s.security.web.FilterChainProxy - Invoking ExceptionTranslationFilter (9/10)
TRACE o.s.security.web.FilterChainProxy - Invoking AuthorizationFilter (10/10)
TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Authorizing SecurityContextHolderAwareRequestWrapper[ org.springframework.security.web.header.HeaderWriterFilter$HeaderWriterRequest@2051dabe]
TRACE o.s.s.w.a.i.RequestMatcherDelegatingAuthorizationManager - Checking authorization on SecurityContextHolderAwareRequestWrapper[ org.springframework.security.web.header.HeaderWriterFilter$HeaderWriterRequest@2051dabe] using AuthorityAuthorizationManager[authorities=[ROLE_USER]]
DEBUG o.s.security.web.FilterChainProxy - Secured POST /api/v1/posts
```

SecurityFilterChain 3개가 모두 잘 동작하고, 이번에 만든 XAuthAuthenticationFilter와 JwtAuthenticationProvider 에서 인증도 잘 동작하고 있음을 알 수 있습니다.

## 제목, 게시글 내용 유효성 검사

이제 제목 또는 게시글에 내용이 없는 경우 400(Bad Request)를 반환하도록 Post 클래스에 유효성 검사를 위한 annotation을 추가합니다.

```java
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Data
@Entity
@Table(name = "posts")
public class Post {

    // 나머지 생략

    @Column(nullable = false)
    @NotBlank
    private String title;

    // MySQL 에서 LONGTEXT 로 지정되며, 최대 값은 4_294_967_295 bytes(4 GB) 입니다.
    @Lob
    @Column(nullable = false, length = 16_777_216)
    @NotBlank
    private String content;

}
```

여기서 @NotBlank를 추가하였는데, @NotBlank는 null과 길이가 0인 문자열 등 공백을 허용하지 않습니다.

### 테스트

다시 테스트를 해보면, 유효성 검사와 관련된 테스트도 모두 통과합니다.

```
PostRequestTests > 유효한 토큰으로 게시글 생성 요청 PASSED
PostRequestTests > 만료된 토큰으로 게시물 생성 요청 PASSED
PostRequestTests > 유효한 토큰으로 `제목`이 비어있는 게시물 생성 요청 PASSED
PostRequestTests > 유효한 토큰으로 `내용`이 비어있는 게시물 생성 요청 PASSED
```

### 유효성 검사 실패 시 메시지 수정

음... 그냥 적당히 할까 하다가 메시지도 수정해줍니다.

@NotBlank annotation 의 기본 메시지는 다음과 같습니다.

```java
String message() default "{jakarta.validation.constraints.NotBlank.message}";
```

해당 경로의 메시지를 찾아가 보면, 아래와 같습니다.

> must not be blank

ValidationMessages.properties 에 커스텀 메시지를 추가합니다.

```properties
post.title.NotBlank.message=게시글 제목을 입력해주세요.
post.content.NotBlank.message=게시글 내용을 입력해주세요.
```



```java
@Column(nullable = false)
@NotBlank(message = "{post.title.NotBlank.message}")
private String title;

@Lob
@Column(nullable = false, length = 16_777_216)
@NotBlank(message = "{post.content.NotBlank.message}")
private String content;
```

## 실제 테스트

테스트 코드로만 돌리는 것 보다는 실제로 눈으로 보는게 재밌으니 Postman 으로 테스트하고 마무리합니다.

회원가입을 하고, 로그인을 해서 JWT를 받습니다.

```json
{
    "tokenType": "jwt",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJsaW12aWsiLCJpYXQiOjE2OTE2NzcyNzMsImV4cCI6MTY5MTc2MzY3Mywic3ViIjoiMSIsInJvbGVzIjpbeyJhdXRob3JpdHkiOiJST0xFX1VTRVIifV0sImVtYWlsIjoibGltdmlrQGxpbXZpay5jb20ifQ.4-VgagzKfzCX8FWnoq6mkgDg_vULv9d89LG3OHJbUOqBmE53jKBpgJPlh2N6gyh6Nnr_psDlUwmHKZQWEJGshw",
    "expiresIn": 86400000
}
```

그리고 X-AUTH-TOKEN 헤더에 설정합니다.

![Postman 에서 X-AUTH-TOKEN을 설정하는 화면](/assets/img/2023-08-10-making-rest-api-with-spring-boot-3-create-a-new-post/01-X-AUTH-TOKEN.png)

제목과 내용이 모두 비어있는 요청을 해봅니다.

![Postman 에서 제목과 내용이 모두 비어있는 요청한 화면](/assets/img/2023-08-10-making-rest-api-with-spring-boot-3-create-a-new-post/02-not-valid-title-content.png)

그리고 제목만 정상적으로 입력해봅니다.

![Postman 에서 내용만 비어있는 요청한 화면](/assets/img/2023-08-10-making-rest-api-with-spring-boot-3-create-a-new-post/03-not-valid-content.png)

모두 정상적으로도 입력해봅니다. 그럼 uri를 Location 헤더에 지정하여 응답해줍니다.

![Postman 에서 정상적인 게시글 추가 요청한 화면](/assets/img/2023-08-10-making-rest-api-with-spring-boot-3-create-a-new-post/04-valid-post.png)

## 끝일까?

글까지 모두 올리고나서 갑자기 테스트 코드에 글이 잘 저장됐는지 검사하는 코드를 작성하지 않았던게 생각납니다.

혹시나 해서 데이터베이스를 확인해보니, user_id 가 저장이 안돼있습니다.

![MySQL Workbench에서 확인한 데이터 저장 상태 화면](/assets/img/2023-08-10-making-rest-api-with-spring-boot-3-create-a-new-post/05-null-userid.png)

## 테스트 코드 추가

유효한 토큰으로 게시글 저장을 요청한 후에 테스트 케이스에 있던 값들과 일치하는지 확인합니다.

```java
Post savedNewPost = postRepository.findById(1L).get();
assertThat(savedNewPost.getId()).isEqualTo(1);
assertThat(savedNewPost.getTitle()).isEqualTo("title");
assertThat(savedNewPost.getContent()).isEqualTo("content");
assertThat(savedNewPost.getCreatedAt()).isBetween(LocalDateTime.now().minusMinutes(5), LocalDateTime.now());
assertThat(savedNewPost.getUpdatedAt()).isBetween(LocalDateTime.now().minusMinutes(5), LocalDateTime.now());
assertThat(savedNewPost.getUserId().getId()).isEqualTo(1);
```

당연하게도 NullPointerException이 발생하여 실패합니다.

## 수정

처음에는 JPA의 문제인가 생각을 했는데, Spring Security 사용이 아직도 미숙한 탓이었습니다.

Controller 에서 `@AuthenticationPrincipal User user` 과 같이 인증된 Principal 을 넘겨주는 annotation을 표시했습니다. 그런데, 앞서 JwtAuthenticationProvider 에서 setDetails 메서드를 이용해서 details 는 지정을 했는데, principal을 지정하지는 않았습니다.

그래서 JwtAuthenticationToken 에서 principal 에대한 setter 를 추가하고,

```java
public void setPrincipal(Object userDetails) {
        this.principal = userDetails;
    }
```

JwtAuthenticationProvider 에서 setDetails 대신 setPrincipal을 요청하도록 수정하였습니다.

```java
@Override
public Authentication authenticate(Authentication authentication)
        throws JwtException, AuthenticationException {

    var xAuthToken = (XAuthAuthenticationToken) authentication;
    var jws = jwtProvider.parse(xAuthToken.getToken());
    var jwtToken = new JwtAuthenticationToken(jws, List.of(new SimpleGrantedAuthority(getAuthorities(jws))));
    jwtToken.setPrincipal(getDetails(jwtToken));
    return jwtToken;
}
```

이래서 테스트를 작성하는구나... 하고 다시 한 번 깨닫는 순간입니다.

## Outro

이번에는 그냥 잘못 코딩하고 수정하는 과정까지 그냥 쭉 써내려갔더니 글이 더 길어졌습니다. 그래서 저도 보고 싶지 않아지는 단점이 있네요...

이번에는 하는 내내 중간중간 일이 생겨서 끊김이 심하니 더 오래걸린 느낌입니다. 팀 프로젝트가 끝나서 이제 이것만 집중하려 했더니, 또 팀 프로젝트 안하면 수료를 안시켜 준다고 합니다.

투덜투덜 하는 이유는... 일찍 내서 가산점을 받는 계획은 모두 엎어지게 되기 때문이죠. 어쩔 수 없이 다른 가산점들을 노리는걸로 해야겠습니다.

## 그냥 참고

Spring Framework 6.1, Spring Boot 3.2 에서는`RestClient`를 사용할 수 있게 된다는 정보를 딴짓하다가 보게됐습니다([링크](https://spring.io/blog/2023/07/13/new-in-spring-6-1-restclient)). `WebClient` 를 대체한다는데, WebClient는 안써봐서 뭔지 궁금해집니다.
