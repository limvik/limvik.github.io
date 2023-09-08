---
layout: post
title: Java로 살펴보는 싱글턴(Singleton) 패턴
categories:
- OOP
tags:
- OOP
- DI
- Java
date: 2023-09-07 14:27 +0900
---
## Intro

면접 대비용으로 CS 면접 책을 샀는데, 책에 나온 내용대로만 말하면 저라도 탈락시키겠다는 생각이 들어 추가로 정리해볼 겸 작성합니다.

이번 글에서는 책의 시작인 `싱글턴(Singleton) 패턴`을 살펴보겠습니다.

## 싱글턴 패턴

### 정의

특정 클래스(Class)의 `인스턴스(Instance)`가 프로그램 내에서 단 `하나`만 존재하도록 보장하는 디자인 패턴

### 간단한 예제

외부에서 추가적으로 인스턴스를 생성하지 못하도록 `private default constructor` 를 사용하는 특징을 볼 수 있습니다.

#### Eager Initialization

객체가 프로그램 시작시 초기화

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

#### Lazy Initialization

요청시 처음 사용될 때 객체 초기화

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

처음에는 당연히 Lazy한게 더 합리적이라고 생각했는데, 초기화 작업이 오래 걸리는 싱글턴 클래스의 경우 필요할 때 로딩하면 `시스템 성능에 영향`을 미치므로 합리적이지 않다는 글을 읽고나서 생각이 짧았음을 알게되었습니다.

이외에도 싱글턴 패턴에서 `멀티 스레드 환경`에 대응하기 위한 여러가지 방식이 있지만, 아직 멀티 스레드 관련 지식이 부족한 관계로 필요하다면 멀티 스레드 내용을 중심으로 싱글턴 패턴과 멀티 스레드를 함께 다루는 글을 작성해 보겠습니다.

### 사용 사례

#### 데이터베이스  연결

싱글턴 패턴의 사용 사례 중 하나로 데이터베이스 연결이 있습니다. 매 요청마다 데이터베이스를 새롭게 연결하지 않고, 데이터베이스와의 연결을 열어두어 자원을 절약합니다.

이전에 Java 를 배우면서 개인적으로 콘솔 프로그램을 만들어볼 때 사용한 적이 있습니다([Github 링크](https://github.com/limvik/simple-flashcards/blob/main/flashcards-app/src/main/java/com/limvik/dao/DatabaseConnection.java)).

데이터베이스로 SQLite3 를 사용하면서, 싱글턴 패턴을 이용하여 데이터베이스 연결을 위한 클래스 인스턴스를 1개만 유지하도록 하였습니다.

```java
package com.limvik.dao;

import java.nio.file.Path;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import org.sqlite.SQLiteConfig;

public class DatabaseConnection {
    private static DatabaseConnection instance;
    private Connection connection;

    private DatabaseConnection() throws SQLException {
        SQLiteConfig config = new SQLiteConfig();
        config.enforceForeignKeys(true);
        String dbPath = Path.of("src", "main", "resources", "db", "flashcards.db").toString();
        connection = DriverManager.getConnection("jdbc:sqlite:" + dbPath, config.toProperties());
    }

    public static DatabaseConnection getInstance() throws SQLException {
        if (instance == null || instance.getConnection().isClosed()) {
            instance = new DatabaseConnection();
        } 

        return instance;
    
    }

    public Connection getConnection() {
        return connection;
    }

    public void closeConnection() {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                System.err.println(e.getMessage());
            }
        }    
    }
}
```

SQLite3는 동시 접근이 안되고, 간단한 Java 콘솔 프로그램이었기 때문에 아주 간단하게 싱글턴 패턴을 사용하였습니다.

실제 환경에서는 여러 사람이 접속해서 데이터베이스 작업을 수행할 수 있도록 여러 Connection을 보유하고 있는 Pool을 만듭니다. 그리고 요청 시 Pool에서 여러 Connection 중 하나를 꺼내 반환합니다.

하지만 이 Connection Pool 을 잘 관리하는 것이 쉬운 일은 아니기 때문에, Spring Boot 를 배우셨다면 한 번쯤 보셨을 `HikariCP` 와 같은 Connection Pool 라이브러리를 이용하게 됩니다.

>[HikariDataSource.java](https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/HikariDataSource.java#L93), [HikariPool.java](https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/pool/HikariPool.java#L154) 등의 클래스에서 getConnection() 메서드를 통해 connection을 얻습니다.

하지만 HikariCP가 `싱글턴 패턴을 이용하는 것은 아니`며, Spring Boot 이용 시 기본 설정인 싱글턴 스코프로 하나의 인스턴스만 만들어 관리할 수는 있습니다.

### 장단점

앞서 마지막에 언급한 것 처럼 인스턴스를 하나로 유지하기 위한 시도를 하더라도, 싱글턴 패턴을 사용하지는 않습니다. 제가봤던 대부분의 싱글턴 패턴 관련 자료에서는 `안티 패턴`이라고 부를 정도입니다.

그렇다면 어떤 장점과 단점이 있어 그러는 것인지 살펴보겠습니다.

`장점`은 프로세스 내에서 인스턴스 생성을 1회만 하니 생성 비용이 절약되고, 1개의 인스턴스만 있으니 메모리 효율이 좋아진다는 것을 간단하게 유추할 수 있습니다.

`단점`은 싱글턴 패턴을 사용하지 않는 중요한 이유가 되므로 각각 정리해 보겠습니다.

#### 단점1. 가독성 감소

일반적으로 `매개변수`를 통해 다른 클래스에 의존하고 있음을 명시적으로 나타낼 수 있습니다. 하지만, 싱글턴 패턴을 사용하는 경우 클라이언트(호출하는 코드) 구현 내에서 인스턴스를 불러오게 되므로 상세하게 살펴봐야만 싱글턴 클래스를 의존하고 있음을 파악할 수 있습니다.

#### 단점2. 객체 지향 원칙 위반

직접 구현을 호출해서 인스턴스를 얻어오므로, 구현이 아닌 추상에 의존해야 한다는 `DIP(Dependency Inversion Principle)`을 위반합니다. 또한 구현을 변경하는 경우 클라이언트에서도 코드 변경이 필요하므로 `OCP(Open Closed Principle)`을 위반하게 됩니다.

```java
// 싱글턴 Logger 클래스
public class Logger {
    private static final Logger instance = new Logger();

    private Logger() {}

    public static Logger getInstance() {
        return instance;
    }

    public void logMessage(String message) {
        System.out.println("Log: " + message);
    }
}

// Client 클래스는 Logger 싱글턴에 직접 의존합니다.
public class Client {
    public void doSomething() {
        // ... some code ...
        Logger.getInstance().logMessage("Action performed in Client");
    }
}

```

#### 단점3. 확장성 감소

두 개 이상의 인스턴스가 필요한 경우, 새로운 클래스를 생성하는 것 외에는 대응 방법이 없습니다.

참고 자료에 나와 있는 예로는 데이터베이스 Connection Pool이 하나만 존재할 때, 느린 SQL만 별도로 처리하는 Connection Pool 을 만들어 확장하려면 싱글턴 패턴 사용 시 확장할 수 없음을 언급하고 있습니다.

#### 단점4. 테스트 용이성 감소

참고 자료에서는 static method를 지원하지 않는 모킹(mocking) 라이브러리가 있고, 싱글턴 클래스에 멤버 변수가 있는 경우 값이 유지되므로 다른 테스트 결과에도 영향에 미치게 되어 테스트 용이성이 감소함을 이야기 합니다.

아직 모킹 라이브러리를 사용해본적이 없고, JUnit 에서는 테스트 결과에 영향 미치지 않게 하기 위한 방법도 많이 있어서 크게 공감은 안되는 것 같습니다. 그래도 뭔가 조치를 해줘야하니 귀찮은 것은 분명합니다.

#### 단점5. 매개변수 설정 불가

다시 단순한 싱글턴 패턴 코드를 보면, 외부에서 인스턴스 생성이 불가능 하도록 하였으므로 매개변수를 통한 초기화가 불가능합니다.

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

물론 전형적인 싱글턴 패턴에서 벗어나 init() 메서드를 만든다던가 하는 방식으로 수정이 가능하기는 합니다.

#### 장단점 정리

| 장점 | 단점 |
|--|--|
| 인스턴스 생성 비용 절약 | 가독성 감소 |
| 효율적인 메모리 사용 | 객체 지향 원칙(DIP, OCP) 위반 |
| - | 확장성 감소 |
| - | 테스트 용이성 감소 |
| - | 매개변수 설정 불가 |

단점의 갯수가 많기는 하지만, 싱글턴 패턴의 장점도 무시하지 못할 만큼 좋아 보입니다. 싱글턴 패턴이 정말 유용했다면, 단점이 있더라도 극복하면서 사용을 했겠지만 단점을 극복할 좋은 대안이 있기 때문에 굳이 사용되지는 않습니다.

## 싱글턴 패턴의 대안

싱글턴 패턴의 대표적인 대안으로 `DI` 그리고 `DI Container`가 있습니다. Java를 사용하시는 분들이라면 사실상 표준이나 다름없는 Spring 의 DI Container를 이용하고 계시겠죠.

무조건 인스턴스를 1개만 유지하는 것은 아니지만, 기본 설정이 싱글턴 스코프 빈으로 관리하게 되어있습니다.

| Scope | Description |
|--|--|
| [singleton](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-singleton) | **`(Default)`** Scopes a single bean definition to `a single object instance` for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring  `ApplicationContext`. |
| [session](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP  `Session`. Only valid in the context of a web-aware Spring  `ApplicationContext`. |
| [application](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a  `ServletContext`. Only valid in the context of a web-aware Spring  `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/scope.html) | Scopes a single bean definition to the lifecycle of a  `WebSocket`. Only valid in the context of a web-aware Spring  `ApplicationContext`. |

출처: [https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)

Spring 사용 시 Field, Setter Injection 등을 사용할 수 있고 일반적으로는 Constructor Injection을 통해서 의존성을 주입하므로, 앞서 언급한 가독성 감소 단점을 해소합니다. 또 추상에 의존할 수 있게 해주므로 DIP, OCP 를 위반하는 단점도 해소합니다. 그리고 다른 단점들도 멤버 변수의 값이 유지되는 문제를 제외하고, Spring을 사용하시는 분들이라면 많이 해소가 된다는 것을 알 수 있습니다.

인스턴스가 1개라서 발생하는 문제 외에는 DI Container를 이용해 많은 부분 해결을 할 수 있습니다. Spring이 싱글턴 패턴의 단점을 해결하자고 나온 것은 아니지만, 싱글턴 패턴을 굳이 사용할 필요 없게 만들어 버렸습니다.

Spring 을 DI Container 로만 사용해도 되고, [Guice](https://github.com/google/guice) 같은 가벼운 DI Container 전용 라이브러리도 있으니 추후에도 싱글턴 패턴은 선택지가 될 수 없을 것 같습니다.

## Outro

다른 객체지향 언어의 경우에도 모두 DI를 하겠지만, Java는 대표적인 프레임워크인 Spring 이 DI Container 로서의 역할을 한다는 점에서 싱글턴 패턴을 그냥 단순히 인스턴스가 1개인 것을 보장하는 패턴으로 넘기기에는 아쉽다고 느꼈던 것 같습니다.

멀티 스레드 환경에서의 문제점들을 요리조리 피해가기는 했는데, Spring 을 사용하면서 피할 수 없는 문제가 될 가능성이 높으니 아마 곧 마주해야하지 않을까 생각됩니다.

## 참고자료

1. [면접을 위한 CS 전공지식 노트](https://product.kyobobook.co.kr/detail/S000001834833)
2. [디자인 패턴의 아름다움](https://product.kyobobook.co.kr/detail/S000202093794)
