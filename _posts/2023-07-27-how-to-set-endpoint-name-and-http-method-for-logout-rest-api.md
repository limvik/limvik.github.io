---
layout: post
title: Logout REST API의 엔드포인트와 HTTP 메서드는 어떻게 지정해야 할까?
categories:
- API
- REST
tags:
- REST
date: 2023-07-27 11:53 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/spring-security-logout-operation/)에서 Spring Security 에서 logout 하는 방법에 대해 간단하게 살펴봤습니다.

이 다음 문제는 REST API 흉내라도 내볼 겸 REST API best practice 를 따라 팀 프로젝트에서 자원(resource)임을 나타내기 위해 엔드포인트를 동사가 아닌 명사로 사용하려 하고 있습니다. 그런데 logout 은 동사이다 보니, 어떻게 해야할지 몰라서 다른 API 문서를 몇개 살펴보고 기존 구현을 수정했습니다.

## 기존 구현

처리와 응답은 이전 글에서 살펴본 대로 수정을 했습니다. 문제는 @DeleteMapping("/token") 입니다. HTTP DELETE 메서드로 token을 삭제하는 것을 의미하도록 구현했습니다.

```java
@DeleteMapping("/token")
public ResponseEntity<Void> logout(HttpServletRequest request) {
    try {
        request.logout();
    } catch (ServletException e) {
        return ResponseEntity.notFound().build();
    }
    return ResponseEntity.noContent().build();
}
```

인증 정보를 제거한다는 것을 강조하기 위해 token 을 삭제하는 의미를 갖도록 API를 작성했습니다. 그런데 이전 글에서 살펴봤듯 session을 invalidate 해야하고, 이외에도 추가적인 작업들이 많이 있다보니, 요청한 것 외에도 숨겨진 작업이 존재하게 됩니다. 위 코드는 token 지워달라고 했는데 logout이 돼버리는 문제가 발생하게 됩니다.

## 다른 API 문서 살펴보기

먼저 어떤 HTTP 메서드를 사용했는지 몇 가지 제품의 API 문서를 살펴봤는데, 다양하게 사용하고 있었습니다.

### 사용한 HTTP 메서드별 API 문서

- GET
	- [Microsoft Operation Manager](https://learn.microsoft.com/en-us/rest/api/operationsmanager/authentication/logout)
	- [ORACLE REST API for Oracle Identity Cloud Service](https://docs.oracle.com/en/cloud/paas/identity-cloud/rest-api/op-oauth2-v1-userlogout-get.html)
- POST
	- [ORACLE REST API for Session Delivery Manager Release 8.1](https://docs.oracle.com/cd/E97664_01/CGBUA/op-versionid-admin-logout-post.html)
- DELETE
	- [IBM Security Guardium Key Lifecycle Manager 3.0.1](https://www.ibm.com/docs/en/sgklm/3.0.1?topic=services-logout-rest-service)

모두 기술력이 뛰어난 회사에서 다 다르게 사용하니 HTTP 메서드는 선택하기 나름이라는 생각이 듭니다.

### 공통점

그리고 공통점이 있는데 엔드포인트 이름을 모두 동사인 logout 을 사용하고 있습니다.

- http://\<Servername>/OperationsManager/signout
- /oauth2/v1/userlogout
- /rest/{versionId}/admin/logout
- https://\<host>:\<port>/SKLM/rest/v1/ckms/logout

signout 같은 약간의 변형이 있기는 하지만, 모두 동사를 사용합니다.

## 수정하기

위에 링크를 모두 첨부하지는 않았지만, 이외에도 여러 API 문서를 살펴본 결과, 명사를 사용해서 신박하게 해결한 API는 찾을 수 없었습니다.

그래서 저도 Best Practice가 절대적인 규칙은 아니므로, 의미가 더 명확한 logout 을 엔드포인트로 수정했습니다. 또 HTTP 메서드의 경우 서버에 있는 데이터를 지우는 것 외에도 비어있는 SecurityContext 를 다시 만들어 저장하는 등의 다른 작업들도 서버에서 수행하므로 POST 방식으로 수정했습니다.

```java
@PostMapping("/logout")
public ResponseEntity<Void> logout(HttpServletRequest request) {
    try {
        request.logout();
    } catch (ServletException e) {
        return ResponseEntity.notFound().build();
    }
    return ResponseEntity.noContent().build();
}
```

별로 글에서 언급하지는 않았지만 Response도 대부분 멋대로라, 수정하지 않았습니다. 기존에는 로그아웃 성공 시에는 더 이상 사용자의 로그인 정보가 존재하지 않는다는 의미에서 No Content 를 사용하고, 실패 시에는 사용자 정보를 찾을 수 없다는 의미에서 Not Found 를 사용했습니다.

## Outro

REST API는 상황에 따라 팀원과 합의하고, 문서화하는게 중요하겠습니다.
