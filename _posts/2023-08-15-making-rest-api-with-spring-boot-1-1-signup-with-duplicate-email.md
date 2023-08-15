---
layout: post
title: Spring Boot로 REST API 만들어보기 (1)-1 중복 이메일 회원가입 처리
categories:
- API
- REST
tags:
- Spring
- JUnit5
- REST
date: 2023-08-15 16:16 +0900
---
## Intro

일곱 가지 과제를 모두 완성은 했는데, 중복 이메일로 회원가입 하면 500(Internal Server Error) 뜨는게 너무 보기 싫어서 추가 수정합니다.

## 요구사항

요구사항은 제가 만들어야겠습니다.

- 중복 이메일로 회원가입 시 클라이언트에 중복된 이메일임을 알려주세요.

이정도로 정리할 수 있겠습니다.

## 중복 이메일 회원가입 개선

### HTTP response status code

일단 클라이언트의 잘못이니 400번대의 상태 코드를 반환해야 합니다.

적합해보이는 것은 409(conflict)인데, 뭔가 자신이 없어서 누군가 같은 목적으로 사용한 사례가 없는지 찾아보니 하나를 찾았습니다([링크](https://is.docs.wso2.com/en/latest/apis/use-the-self-sign-up-rest-apis/#/Self%20Register/post_me)).

GPT 한테 한 번 물어보기도 하고, 시간이 많이 없으니 빠르게 409(conflict)로 결정합니다.

### 테스트 코드 작성

동일한 이메일로 두 번의 회원가입을 요청하여 409(Conflict) 반환 그리고, 오류 메시지가 body에 포함되어 있는지 확인합니다. 마무리로 데이터베이스에는 가입된 회원수가 변화 없음을 확인합니다.

```java
@Test
@DisplayName("중복된 이메일로 가입 요청")
void shouldNotCreateANewUserIfDuplicateEmail() {

    requestCreateANewUser();
    ResponseEntity<String> createResponse = requestCreateANewUser();

    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CONFLICT);
    assertThat(createResponse.getBody().toString()).contains("중복된 이메일 입니다.");

    assertThat(userRepository.count()).isEqualTo(1);
}

private ResponseEntity<String> requestCreateANewUser() {
    return restTemplate.postForEntity("/api/v1/user", user, String.class);
}
```

### 테스트

테스트를 해보면 예상대로 결과가 나옵니다.

```
expected: 409 CONFLICT
 but was: 500 INTERNAL_SERVER_ERROR
```

### Controller 수정

어떤 예외가 던져지는지 직접적으로 출력이 안돼서 디버거로 확인해본 결과 DataIntegrityViolationException 이 던져집니다.

```java
@PostMapping
private ResponseEntity<String> signUp(@Valid @RequestBody User newUser,
                                    UriComponentsBuilder ucb) {
    User savedUser;
    try {
        savedUser = userService.createUser(newUser);
    } catch (DataIntegrityViolationException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body("중복된 이메일 입니다.");
    }

    URI locationOfNewUserAccount = buildLocationOfNewUserAccountUri(savedUser, ucb);
    return ResponseEntity.created(locationOfNewUserAccount).build();
}
```

### 테스트

다시 테스트를 해보면, PASSED가 됩니다.

```
중복된 이메일로 가입 요청 PASSED
```

## Outro

다행히 간단하게 끝났습니다. 그럼 문서화 작업을 마저하러 가야겠습니다.
