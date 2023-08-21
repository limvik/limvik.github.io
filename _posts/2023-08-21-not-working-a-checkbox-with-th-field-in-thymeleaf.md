---
layout: post
title: Thymeleaf 에서 th:field 속성 사용 시 checkbox가 정상동작하지 않는 경우 해결책
categories:
- etc
tags:
- Spring
- Thymeleaf
date: 2023-08-21 11:05 +0900
---
## Intro

이전 글([링크](/posts/can-not-find-setter-of-record-class-in-mybatis/))과 마찬가지로 강제당한 팀  프로젝트에서 Thymeleaf 를 사용 중입니다.

## 해결방법

`application.properties` 파일을 사용하시는 경우 아래와같은 설정을 추가해줍니다.

```properties
spring.thymeleaf.render-hidden-markers-before-checkboxes=true
```

## 상황

아래와 같이 bootstrap 을 사용하고 있으며, Thyemeleaf 를 이용하여 체크박스 여러개를 만들어주고 있습니다.

```html
<th:block th:each="benefit : ${benefits}" th:object="${postForm}">
    <input th:type="checkbox" class="btn-check" th:id="|benefit${benefit.id()}|" autocomplete="off" th:field="*{benefits}" th:value="${benefit.id()}">
    <label class="btn btn-outline-secondary" th:for="|benefit${benefit.id()}|" th:text="${benefit.name()}"></label>
</th:block>
```

이 코드를 통해서 체크박스에 체크가 된 값들을 `List`로 받기 위해 `th:field` 속성을 사용하였는데 체크박스로 동작하지 않습니다.

## 원인

위 코드 그대로 실행을 해보면 아래와 같은 방식으로 렌더링 됩니다.

```html
<input type="checkbox" class="btn-check" id="benefit6" autocomplete="off" value="6" name="benefits">
<input type="hidden" name="_benefits" value="on">
<label class="btn btn-outline-secondary" for="benefit6">야근 교통비 지원</label>
```

`hidden 타입의 input 태그`가 사이에 끼어드는 것을 볼 수 있습니다. 체크박스 타입의 input 태그 뒤에 바로 label 태그가 와야 정상적으로 동작하는데, 갑자기 hidden 타입의 input 태그가 끼어들어 정상적으로 동작하지 않는 것입니다.

이에 대한 설명은 [Issue 링크](https://github.com/thymeleaf/thymeleaf-spring/issues/178)를 참고하시면 됩니다.

## 해결 방법 및 결과

제일 앞에서 언급드린대로 properties 를 추가하면 됩니다.

```properties
spring.thymeleaf.render-hidden-markers-before-checkboxes=true
```

그러면 직관적인 속성 이름과 같이 hidden 타입의 input 태그가 먼저 오게되고 정상적으로 동작하는 것을 확인할 수 있습니다.

```html
<input type="hidden" name="_benefits" value="on">
<input type="checkbox" class="btn-check" id="benefit1" autocomplete="off" value="1" name="benefits">
<label class="btn btn-outline-secondary" for="benefit1">야근 없음</label>
```
