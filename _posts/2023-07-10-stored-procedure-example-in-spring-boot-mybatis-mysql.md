---
layout: post
title: Spring Boot, MyBatis, MySQL Stored Procedure 사용 예제
categories:
- Database
- MySQL
tags:
- Spring Boot
- MyBatis
- MySQL
- Stored Procedure
date: 2023-07-10 21:19 +0900
---
## Intro

팀 프로젝트에 MyBatis와 MySQL을 사용중입니다. 오늘 Stored Procedure 를 사용하려는데, 값을 파라미터로 보내는 예제는 많은데 출력해서 값을 가져오는 건 쓸만한 예제가 잘 안보입니다.

그래서 문서롤 보니 MyBatis 문서는 개인적으로 좀 난잡한거 같아서 GPT에 물어봤습니다. 인터넷에 괜찮은 예제가 많이 없어서 그런가 GPT-3.5는 헛소리를 많이 하고, GPT-4 한테 물어보니 바로 괜찮은 예제를 줬습니다.

공유하고 나중에 저도 다시 참고할 겸 올립니다.

## Stored Procedure 예제

예제는 DELETE 문 실행 후 삭제한 행의 수를 받아오는 예제입니다. 다 아시겠지만 혹시나 해서 말씀드리면, DELETE 문 하나만 실행해서 삭제한 행의 수를 받아오는 것은 그냥 DELETE 문 사용하면 됩니다. 저 같은 경우는 여러 SQL 실행 후에 DELETE 문 사용하려고 Stored Procedure를 사용했습니다.

설명이 딱히 길지는 않아서 GPT-4가 준 예제 그대로 붙여넣겠습니다.

### GPT-4가 준 예제

먼저, MyBatis XML 매퍼 파일과 관련하여 MySQL의 DELETE 문에서 삭제된 행의 수를 반환하는 방법을 보여드리겠습니다.

MySQL에는 간단한 DELETE 문을 사용하면 실행된 DELETE 작업으로 인해 영향을 받은 행의 수를 반환하는 기능이 있습니다. 이를 사용하면 MyBatis의 반환 값으로 이를 가져올 수 있습니다.

1. 먼저 Stored Procedure를 생성해 봅시다.

```sql
DELIMITER //
CREATE PROCEDURE deleteData(IN inId INT, OUT deletedRowCount INT)
BEGIN
   DELETE FROM yourTable WHERE id = inId;
   SET deletedRowCount = ROW_COUNT();
END //
DELIMITER ;
```

이 Stored Procedure는 'inId' 매개변수로 받은 ID에 해당하는 레코드를 'yourTable' 테이블에서 삭제합니다. 그리고 `ROW_COUNT()` 함수를 사용하여 삭제된 행의 수를 'deletedRowCount' 변수에 할당합니다.

2. 그런 다음 MyBatis의 XML 매퍼 파일에서 이 Stored Procedure를 호출합니다.

```xml
<mapper namespace="com.yourpackage.YourMapper">
    <select id="deleteData" parameterType="map" statementType="CALLABLE">
        {call deleteData(#{inId, mode=IN, jdbcType=INTEGER}, #{deletedRowCount, mode=OUT, jdbcType=INTEGER})}
    </select>
</mapper>
```

3. 마지막으로 Java 코드에서 MyBatis 매퍼를 사용하여 이 메소드를 호출합니다.

```java
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

@Component
public class YourDao {
    private final SqlSessionFactory sqlSessionFactory;

    @Autowired
    public YourDao(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }

    public int deleteData(int id) {
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            Map<String, Object> params = new HashMap<>();
            params.put("inId", id);
            sqlSession.selectOne("com.yourpackage.YourMapper.deleteData", params);
            return (int) params.get("deletedRowCount");
        }
    }
}
```

이렇게 하면 `deleteData` 메소드는 삭제된 행의 수를 반환하게 됩니다. 이것이 원하시는 결과에 부합하는지 확인해 보세요.

---

저는 SqlSession 을 사용하지 않아서 예제와는 달리 @Repository와 @Mapper가 표시된 repository bean의 메서드로 params 를 넘겨서 사용했습니다. 간단한 예제지만 사용하는 방법은 잘 포함하고 있어서 큰 무리 없이 변형해서 사용하실 수 있을거라 생각됩니다.

## 소소한 관련 정보

### jdbcType

jdbcType 문서 설명 번역 - 이 표 다음에 나오는 지원되는 type 목록의 JDBC type입니다. JDBC 유형은 INSERT, UPDATE 또는 DELETE 시 nullable한 열에만 필요합니다. 이는 MyBatis 요구 사항이 아니라 JDBC 요구 사항입니다. 따라서 JDBC를 직접 코딩하는 경우에도 이 유형을 지정해야 하지만 nullable 한 값에 대해서만 지정해야 합니다.

MyBatis가 지원하는 jdbcType 목록([링크](https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#%EC%A7%80%EC%9B%90%EB%90%98%EB%8A%94-jdbc-%ED%83%80%EC%9E%85))

MyBatis 문서는 한글이 더 어려워서, 번역기로 번역한 글을 넣었습니다.

위에서 nullable 한 값에 대해서만 지정해야 한다고 적혀있는데, 예제에서는 Map을 사용하여 모두 nullable하니 jdbcType을 지정해주는게 오류를 줄이는 길인 것 같습니다.

INTEGER type인 mode가 out인 parameter에 jdbcType을 지정하지 않으면 아래와 같은 예외가 던져집니다.

```
org.apache.ibatis.executor.ExecutorException: The JDBC Type must be specified for output parameter.  Parameter: deletedRowCount
```

### 직접 Stored Procedure 호출하기

테스트 삼아 직접 Stored Procedure를 호출해보려고 하는데, 출력 파라미터는 어떻게 줘야할지 잘 몰랐었습니다. 그래서 저와 같은 분이 있을거라 생각돼서 참고삼아 붙여넣습니다.

```sql
CALL deleteData(123, @deletedRowCount);
SELECT @deletedRowCount;
```

### Stored Procedure 의 주요 장점과 단점

데이터베이스 이론 공부할때 책에서 Stored Procedure 가 보안에 좋고, 성능이 좋다는 식으로 봤던 기억이 있어 가볍게 훑어볼겸 GPT-4에 물어봤습니다.

---

Stored Procedure의 주요 장점과 단점을 살펴보면 다음과 같습니다:

장점:
1. **프리 컴파일:** Stored Procedure는 한 번 컴파일되고, 그 후에는 계속 재사용됩니다. 이러한 특징은 복잡한 쿼리의 경우 시간을 절약할 수 있습니다.
2. **네트워크 트래픽 감소:** Stored Procedure는 클라이언트와 서버 간의 통신을 줄일 수 있습니다. 단일 Stored Procedure 호출로 여러 SQL 문을 실행할 수 있기 때문입니다.
3. **보안:** 특정 Stored Procedure에만 접근 권한을 제공하여 데이터에 대한 접근을 제한할 수 있습니다.

단점:
1. **유지 관리:** Stored Procedure를 변경하려면 데이터베이스에서 직접 변경해야 합니다. 이는 응용 프로그램 코드와 별도로 유지 관리해야 하는 로직을 만들게 됩니다.
2. **포팅성:** Stored Procedure는 일반적으로 특정 데이터베이스에 특화되어 작성됩니다. 이로 인해 다른 데이터베이스 시스템으로 이동하는 것이 어려울 수 있습니다.
3. **디버깅:** 많은 데이터베이스 시스템에서 Stored Procedure의 디버깅이 복잡하거나 제한적일 수 있습니다.

따라서, Stored Procedure가 성능 향상을 가져다 주는 경우도 있지만, 애플리케이션의 요구 사항, 데이터베이스 구조, 그리고 개발팀의 기술 능력에 따라 그 효과는 다르게 나타날 수 있습니다. 특히 성능 이점을 얻기 위해 Stored Procedure를 사용하는 것이 항상 최선의 방법이 아닐 수 있으며, 애플리케이션 로직의 복잡성, 유지 관리 노력, 그리고 데이터베이스 간 이식성 등을 고려해야 합니다.

---

시험 공부용으로 Stored Procedure는 `프리 컴파일(precompile)`이 키워드 였는데, 보니까 생각났습니다.

단점 중에 포팅성, 저한테 친숙한 단어로 보자면 이식성은 MyBatis 쓰면 사실 이식성은 어느정도 포기한거라 생각돼서 Stored Procedure를 적극 사용해도 괜찮지 않나 싶습니다. 하지만 아직 아는게 없어서 그럴 수 있으니 단정 짓지는 말아야겠습니다.

## Outro

관련 정보를 다른데 두면, 어디 뒀는지도 잊어버릴거 같아 그냥 다 붙여넣었는데, 도움이 되셨기를 바랍니다!
