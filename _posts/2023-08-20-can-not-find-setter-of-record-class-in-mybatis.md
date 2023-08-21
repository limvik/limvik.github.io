---
layout: post
title: MyBatis에서 record 클래스 사용 시 setter 관련 오류 및 해결방법
categories:
- etc
tags:
- MyBatis
- Java
date: 2023-08-20 21:39 +0900
---
## Intro

회고할 틈도 없이 바로 또 팀 프로젝트를 강제당해서(?) 작업 중입니다. 짧은 기간의 프로젝트라 팀원들을 위해, 모두가 사용해본 MyBatis를 사용 중 입니다.

최근에 record 클래스에 맛을 들여서 DTO(Data Transfer Object)는 특별한 이유가 있지 않은 이상 record 클래스를 사용하는 편입니다. 그런데 평소대로 MyBatis를 사용하니 record 클래스에서 setter 를 못찾는다고 오류를 내뱉습니다.

## 상황

저는 아래와 같이 record 클래스로 간단하게 선언을 한 상황입니다.

```java
package com.juniorok.juniorok.dto;

public record Benefit(long id, String name) { }
```

평소 XML mapper 에는 아래와 같이 선언해서 사용합니다.

```xml
<resultMap id="benefitResultMap" type="com.juniorok.juniorok.dto.Benefit">
    <id property="id" column="id" />
    <result property="name" column="name" />
</resultMap>


<select id="findAllBenefitTags" resultMap="benefitResultMap">
    SELECT * FROM benefits;
</select>
```

그리고 이대로 사용하면, 오류가 발생합니다.

## 오류 메시지

아래와 같이 `Benefit` 이라는 클래스에서 `id` 라는 이름의 프로퍼티를 위한 setter 가 없다고 합니다.

> org.apache.ibatis.reflection.ReflectionException: There is no setter for property named 'id' in 'class com.juniorok.juniorok.dto.Benefit'  

## Release Note

MyBatis 가 record 클래스를 아직 지원을 안하나 싶어서 Release Note([링크](https://github.com/mybatis/mybatis-3/releases/tag/mybatis-3.5.10))를 찾아보니, `3.5.10`에서 수정이 됐습니다.

> `IllegalAccessException`  when auto-mapping Records (JEP-359)  [#2195](https://github.com/mybatis/mybatis-3/issues/2195)  

제가 본 Exception 과는 다른 Exception 이지만, 같은 문제로 보입니다.

## Pull Request

위에 적힌 Issue 에서 [PR 링크](https://github.com/mybatis/mybatis-3/pull/2477/files)를 따라 들어가 테스트 쪽을 살펴봅니다.

그러면 아래와 같이 record 클래스를 사용하고 있는 것을 볼 수 있습니다.

```java
// Property.java
public record Property(int id, String value, String URL) {
  public String value() {
    // Differentiate between method call and field access
    return value + "!";
  }
}

// RecordTypeMapper.java
@Results(id = "propertyRM")
@Arg(column = "id", javaType = int.class)
@Arg(column = "val", javaType = String.class)
@Arg(column = "url", javaType = String.class)
@Select("select val, id, url from prop where id = #{id}")
Property selectProperty(int id);
```

## 수정하기

테스트에 작성된 코드를 따라서 코드를 아래와 같이 수정하였고, 이상 없이 출력됨을 확인하였습니다.

```java
// Benefit.java
package com.juniorok.juniorok.dto;

public record Benefit(long id, String name) { }

// Repository
@Results(id = "benefitResultMap")
@Arg(column = "id", javaType = long.class)
@Arg(column = "name", javaType = String.class)
@Select("SELECT * FROM benefits")
List<Benefit> findAllBenefitTags();
```

## 추가 확인 - XML은 안됨

SQL은 모두 XML 파일에 모아두고 있어서, 통일하기 위해 XML로도 실험을 해봤는데 안됩니다. 처음과 마찬가지로 setter를 찾지 못합니다.

```xml
<resultMap id="benefitResultMap" type="com.juniorok.juniorok.dto.Benefit">
    <constructor>
        <idArg column="id" javaType="_long" name="id" />
        <arg column="name" javaType="String" name="name" />
    </constructor>
    <id property="id" column="id" />
    <result property="name" column="name" />
</resultMap>

<select id="findAllBenefitTags" resultMap="benefitResultMap">
    SELECT * FROM benefits;
</select>
```

> XML \<constructor> 문서([링크](https://mybatis.org/mybatis-3/sqlmap-xml.html#constructor))

## Outro

XML 에서도 되는 방법을 찾아보고 싶지만, 과연 이 프로젝트를 끝나고 MyBatis 를 사용할 일이 있을지 잘 모르겠어서 해결 방법만 찾아보고 넘어가기로 했습니다.