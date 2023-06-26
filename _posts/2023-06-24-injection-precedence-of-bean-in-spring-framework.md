---
layout: post
title: Spring Framework 에서의 Bean 주입 우선순위
categories:
- Framework
- Spring
tags:
- Spring
- Java
date: 2023-06-24 23:58 +0900
---
## Intro

이전에 Spring을 간단히 배우고서 API를 `되기는 한다` 라는 느낌으로 만들어봤습니다. 다음 주에 시작할 팀 프로젝트를 할 때는 RESTful API에 `발은 걸쳐봤다` 라는 느낌은 받고 싶어 Spring을 이용해서 RESTful API를 만드는 방법에 대해 공부를 시작했습니다.

Spring을 이용해서 API를 만드는게 주 목적인 책([링크](https://www.packtpub.com/product/modern-api-development-with-spring-and-spring-boot/9781800562479))을 보고 있는데, 저자분이 API를 만들기 위한 Spring 기본만 다루겠다며 기본적인 지식을 설명해줍니다. 근데 기본서에서는 본적없는 @Primary 같은 annotation과  Autowiring 시 주입되는 Bean의 우선순위에 대해 이야기를 합니다.

기본을 익힐 수 없는 기본서를 봤나봅니다. 그래서 우선순위에 대해 정리해볼 겸 글을 작성해봅니다.

## 내가 사용하는 우선순위 결정 방식

강의나 기본서에서 배운 방식은 여러 클래스가 같은 상위 interface를 사용하는 경우 주입되는 구현 클래스를 특정하기 위해 주로 `@Qualifier` 를 사용했습니다. 딱히 우선순위를 고려한다기 보다는 오류나지 않게 하려고 썼었습니다.

공통의 interface 를 생성하고, 다르게 구현한 두 클래스에 @Qualifier를 이용하여 이름을 지정합니다. 예제는 Spring Boot 3.1.1을 사용합니다.

```java
public interface MessageService {
    String getMessage();
}

// 첫 번째 구현체
import org.springframework.stereotype.Service;

@Service("simple")
public class SimpleMessageService implements MessageService {

    @Override
    public String getMessage() {
        return "Hello World!";
    }

}

// 두 번째 구현체
import org.springframework.stereotype.Service;

@Service("advanced")
public class AdvancedMessageService implements MessageService {

    @Override
    public String getMessage() {
        return "Hello API World!";
    }

}
```

그리고 의존성을 주입할 클래스를 생성합니다. 사실 이전에는 맨날 field injection만 하다가 리플렉션 공부하면서 constructor injection을 써야겠다 싶어서 Spring에서는 처음 써봅니다. 그래서 빈으로 등록만 하면 @Autowired 뿐만아니라 심지어 매칭되는게 1개 라면 @Qualifier도 안써도 된다는 사실을 처음 알았습니다. reflection 공부했던 것을 생각해보면, bean으로 등록된 클래스들은 Class 클래스를 불러와서 field, constructor, method를 탐색하고 bean 중에 일치하는게 있는지 모두 확인하여 injection 까지 하는 걸로 예상됩니다.

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import com.limvik.precedence.service.MessageService;

@Component
public class MessagePrinter {
	
    private MessageService messageService;
	
    MessagePrinter(@Qualifier("advanced") MessageService messageService) {
        this.messageService = messageService;
    }
	
    public void printMessages() {
        System.out.println(messageService.getMessage());
    }
	
}
```

마지막으로 printMessages 메서드를 메인에서 호출해줍니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AnnotationPrecedenceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AnnotationPrecedenceApplication.class, args)
            .getBean(MessagePrinter.class)
            .printMessages(); // Hello API World!
    }

}
```

이전에 MessagePrinter 클래스에서 @Qualifier를 advanced로 지정했기 때문에 `Hello API World!`가 출력되는 것을 볼 수 있습니다.

아래와 같이 MessagePrinter에서 생성자에 @Qualifier 없이 @Autowired만 사용한다면, Spring에서 어떤 것을 사용해야 할지 몰라 오류가 발생합니다.

```java
@Autowired
MessagePrinter(MessageService messageService) {
    this.messageService = messageService;
}
```
굳이 이걸로는 일부러 오류를 발생시켜보지는 않았더니, 오류 메시지를 처음 봤습니다. 책에서 처음 봤던 @Primary 혹은 @Qualifier 를 사용하라는 action 메시지도 있습니다.
```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.limvik.precedence.MessagePrinter required a single bean, but 2 were found:
	- advanced: defined in file [C:\kdt\workspace_boot\annotation-precedence\target\classes\com\limvik\precedence\service\AdvancedMessageService.class]
	- simple: defined in file [C:\kdt\workspace_boot\annotation-precedence\target\classes\com\limvik\precedence\service\SimpleMessageService.class]


Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

한쪽에 @Primary 를 표시하면 @Primary를 표시한 클래스의 message가 출력되는 것을 볼 수 있습니다. 굳이 코드를 포함하지는 않겠습니다.

constructor injection을 처음 사용해보긴 했지만, 이런 흐름으로 평소에 사용했습니다.

먼저 우선순위에 대해 다른 좋은 글을 발견해서 훑어봤습니다.

## 다른 글 참고해보기

카카오페이 기술블로그([링크](https://tech.kakaopay.com/post/martin-dev-honey-tip-2/))에 Spring 내부 코드까지 분석해주신 분이 있어 직접 요약해주신 내용만 가져와 봤습니다. 자세한 내용은 링크를 확인하시면 됩니다.

### 요약

- @Bean 으로 bean을 생성하게 되면, method name이 bean name으로 생성된다.
- 같은 Type의 bean이 1개만 있다면, bean name과 관련없이 bean을 주입해준다.
- 같은 Type의 bean이 여러 개 있으면, @Qualifier가 없어도 bean name과 field name을 매칭해서 bean을 주입해준다.
- (!) @Primary가 있으면, bean name을 무시하고 Type 기반으로 Primary인 Bean을 주입한다.
- @Qualifier가 있으면, 무조건 bean name 기준으로 주입해준다. (없으면 오류가 발생한다)
- @Qualifier 어노테이션이 @Primary 어노테이션보다 우선하여 적용된다.

지금 보고 있는 책에서는 Autowiring 시 우선순위를 type, qualifier, name 순으로 제시하고 있습니다. 위 요약 결과와 대조해보면 type을 우선 고려하고 type으로 판단 불가능할 때 qualifier로 판단하고, qualifier가 없다면 name 으로 판단한다는 것을 알 수 있습니다.

그리고 책에서 @Primary가 default 를 설정하는데 사용한다고 표현했는데, @Primary 가 설정되면 bean name도 무시하는 것을 봐서는 default 라고 하기에는 어려운 것 같습니다.

어느 정도 정리가 되고나니 쓸데없는 호기심이 발동해서 여러 개 있을 때도 같은 type을 잘 찾아서 주입해주는지 궁금해집니다.

## 다른 type 간에 Spring Autowiring 우선순위 확인해보기

다른 type이 여러 개 있을 때도 같은 type을 잘 찾아서 주입해주는지 확인해 보겠습니다.

### 다른 type, 같은 qualifier 와 name

matching 되는 type이 있다면 해당 type에서 qualifier와 name을 찾아서 주입해주지 않을까? 하는 막연한 생각으로 시작해 봅니다.

다른 type을 만들어주기 위해 interface 하나와 구현체 하나를 만듭니다. 이때 구현체에 이전에 다른 구현체에서 사용했던 advanced로 중복된 name을 지정해줍니다.

```java
// 인터페이스
public interface LimvikService {
    String getMessage();
}

// 구현체
@Service("advanced")
public class SeahorseMessageService implements LimvikService {

    @Override
    public String getMessage() {
        return "Seahorse is comming";
    }

}
```

그러면...! 에러가 납니다. 너무 길어서 주요 메시지만 가져와 보면 아래와 같습니다.

```
Failed to parse configuration class
...
Annotation-specified bean name 'advanced' for bean class 
[com.limvik.precedence.service.SeahorseMessageService] conflicts with existing, non-compatible bean definition of same name and class [com.limvik.precedence.service.AdvancedMessageService]
```

자 이렇게 애초에 발생할 일이 없을 우선순위 문제를 테스트 해봤습니다. `'아무리 다른 type의 bean이라도 동일한 bean name을 사용할 수 없다.'` 메모... 혹시나 하면서 다른 package로 옮겨봤지만 안됩니다. `type에 따라 bean name이 격리될 거라고 잘못 생각`하고 있었던 것 같습니다.

그럼 이번에는 Configuration 파일을 이용해서 bean을 등록해 봅니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.limvik.precedence.service.AdvancedMessageService;

@Configuration
public class AppConfig {
    @Bean
    public AdvancedMessageService messageService() {
        return new AdvancedMessageService();
    }
}
```

그리고 동일한 메서드 이름을 갖는 LimvikConfig를 하나 더 추가합니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.limvik.precedence.limvikservice.SeahorseMessageService;

@Configuration
public class LimvikConfig {
    @Bean
    public SeahorseMessageService messageService() {
        return new SeahorseMessageService();
    }
}
```

둘 다 따로 bean name을 별도로 지정하지 않았으므로, 메서드 이름인 messageService가 bean name이 됩니다.

기존의 구현체에서 @Service annotation은 모두 제거해 주고(코드 생략), MessagePrinter에서는 @Qualifier 로 messageService를 지정해줍니다.

```java
@Component
public class MessagePrinter {
	
    private MessageService messageService;
	
    MessagePrinter(@Qualifier("messageService") MessageService messageService) {
        this.messageService = messageService;
    }
	
    public void printMessages() {
        System.out.println(messageService.getMessage());
    }
	
}
```

두 개의 다른 Type의 bean name이 같을 때, 해당 bean name으로 Qualifier를 지정하면 어떻게 될지 확인합니다.

해당 타입이 주입되는지 확인하기 위해 실행해 보면...! 에러가 납니다!

```
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'messageService', defined in class path resource [com/limvik/precedence/LimvikConfig.class], could not be registered. A bean with that name has already been defined in class path resource [com/limvik/precedence/AppConfig.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

그런데... @Service에 같은 이름을 지정했을 때와는 action 메시지가 좀 다릅니다. component는 엄밀히 따지면 bean이 아닌가? 내가 잘못알고 있나? 하는 생각이 들어 다시 앞의 메시지를 살펴보면 `Annotation-specified bean name 'advanced' for bean class` 라는 내용이 있는 것을 보니 bean 이라고 언급하고 있습니다. 그럼 @Bean annotation을 사용할 때와 @Component annotation을 사용할 때 처리하는게 달라서 그런가 본데... 일단은 지금 하고 있는 우선순위 실험을 끝내는데 집중해야겠습니다.

action 메시지를 따라서 application.properties 파일에 bean definition overriding 이 가능하도록 설정해줍니다.

그리고 실행 해보면...! 또 에러가 납니다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.limvik.precedence.MessagePrinter required a bean of type 'com.limvik.precedence.service.MessageService' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Qualifier("messageService")


Action:

Consider defining a bean of type 'com.limvik.precedence.service.MessageService' in your configuration.
```

이번에는 MessageService 를 찾지 못합니다. 구현체로 Type을 지정하면 찾을까 싶어서 AdvancedMessageService로 Type을 변경해봤지만 동일한 에러가 발생합니다.

혹시나해서 앞서 `패키지 위치를 변경`했던 것 처럼, AppConfig의 패키지 위치를 변경해서 LimvikConfig와 `서로 다른 패키지에 위치시켰더니 정상적으로 동작`합니다. 이건 또 왜 되는지 Spring 코드가 또 궁금해지지만, 지금은 그럴 시간이 없기 때문에 차후 과제로 남겨두겠습니다.

같은 bean name을 쓰고 싶은 경우 @Component annotation을 사용하면 안되고, Configuration 파일을 만들고 Configuration 파일은 다른 패키지에 위치시켜야 한다고 정리해볼 수 있겠습니다.

### 다른 type, 같은 qualifier, 다른 name

그러면 다시 같은 패키지로 이동시킨 다음에 한 가지 실험을 더 해봅니다. 주입되어야 할 bean name을 한쪽만 간절한 마음을 담아 다르게 지정합니다.

```java
@Configuration
public class AppConfig {
    @Bean("please")
    public AdvancedMessageService messageService() {
        return new AdvancedMessageService();
    }
}
```

이번에도 에러가 나지만 메시지가 조금 더 바뀌었습니다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.limvik.precedence.MessagePrinter required a bean of type 'com.limvik.precedence.service.MessageService' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Qualifier("messageService")

The following candidates were found but could not be injected:
	- User-defined bean method 'messageService' in 'AppConfig'


Action:

Consider revisiting the entries above or defining a bean of type 'com.limvik.precedence.service.MessageService' in your configuration.
```

candidates를 찾기는 했는데, 주입은 못하겠답니다. 뭐지...?

그럼 그냥 Qualifier를 지워봤더니, 실행이 잘 됩니다.

```java
@Component
public class MessagePrinter {
	
    private MessageService messageService;
	
    MessagePrinter(MessageService messageService) {
        this.messageService = messageService;
    }
	
    public void printMessages() {
        System.out.println(messageService.getMessage());
    }
	
}
```

이미 우선순위는 뭔가 안맞는 것으로 판명이 됐지만, 엉뚱한 qualifier 를 적어주면 name 을 찾아서 주입해주지 않을까? 하는 호기심이 생깁니다.

하나만 명확하게 주입대상이 되도록 LimvikConfig는 @Configuration을 지워버리고 AppConfig에 설정된 Bean은 설정된 이름을 제거하여 bean name이 messageService가 되게합니다.

```java
public class LimvikConfig {
    @Bean
    public SeahorseMessageService messageService() {
        return new SeahorseMessageService();
    }
}

@Configuration
public class AppConfig {
    @Bean
    public AdvancedMessageService messageService() {
        return new AdvancedMessageService();
    }
}
```

그리고 생성자에서 @Qualifier에는 다른데 없는 limvik 이라는 문자열을 지정합니다.

```java
MessagePrinter(@Qualifier("limvik") MessageService messageSericve) {
    this.messageService = messageSericve;
}
```

그러면 앞서 봤던, 알지만 injection은 못한다는 에러가 발생합니다.

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.limvik.precedence.MessagePrinter required a bean of type 'com.limvik.precedence.service.MessageService' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Qualifier("limvik")

The following candidates were found but could not be injected:
	- User-defined bean method 'messageService' in 'AppConfig'


Action:

Consider revisiting the entries above or defining a bean of type 'com.limvik.precedence.service.MessageService' in your configuration.
```

내부 코드가 파악 되지 않은채로 실험을 더 해보는게 더 이상은 의미가 없는 것 같아 여기서 중단합니다.

## 정리해보기

1. 책에서 이야기한 type, qualifier, name 우선순위는 먼저 type을 식별하고, 해당 type에 주입 가능한 class가 여러 개 일때 qualifier가 있으면 qualifier로, qualifier가 없으면 name으로 판단하여 의존성 주입하는 것을 의미
2. @Primary 는 @Qualifier 보다는 우선순위가 낮지만, name 보다는 우선순위가 높음
3. @Component 사용 시 패키지 상관 없이 중복 bean name이 있으면 무조건 오류가 발생하지만, @Bean 사용 시에는 properties 파일에 bean name 중복이 가능하게 설정해주면 다른 패키지에 Configuration 파일을 위치시켜 정상적으로 사용이 가능

혼란 스럽지 않게 하려면 우선순위를 따지기보다는 bean name이 중복되지 않도록 주의하고, @Qualifier 가 지정되면 type이고 뭐고 가장 먼저 고려한다는 사실을 기억하는게 좋겠습니다.

### 먼 훗날 해결할 질문

1. @Bean 을 사용하면, 같은 bean name이 있는데도 왜 package를 달리하면 정상 작동(properties에 bean definition overriding 설정된 상태)할까?
2. bean definition overriding 이 bean name 중복 외에 다른 무언가가 있는걸까?
3. 실험 방식 자체가 잘못된건 아닐까?

## Outro

더 해보고 싶은 실험도 있고, 다른 type에 대한 우선순위는 재미삼아 해보면서 조건들을 아무 생각 없이 바꾸다보니 너무 혼란스러워 중단했습니다. 내부 코드도 모른채로 하면 더 이상 실익이 없는 것 같기도 하고...

아쉬움이 남기는 하지만 @Component 를 사용할 때와 @Bean 을 사용할 때 동작하는 방식이 다른게 흥미롭기는 했습니다. 그냥 다 같은 로직으로 DI Container를 향해 돌진해서 Bean이 될 줄 알았는데 말입니다. 코드를 뜯어봐야 자세히 알 수 있을 것 같습니다.

중복된 bean name을 사용할 여지를 주지 않으려면 @Component를 사용하는게 더 맞는 것 같기는 하지만, @Bean 사용하면서 굳이 설정파일까지 바꿔가며 중복된 bean name을 사용할 개발자는 없을 것 같기는 합니다.

프로그래밍 언어 사용 시에 모든 연산자 우선순위를 외우기 힘들어서 괄호 사용하듯이 어떤 Bean이 주입될지 명확하게 만드는게 제일 중요할 것 같습니다. 개인적으로는 @Component 사용해서 중복된 이름이 사용될 여지를 주지 않고, @Qualifier를 사용하는게 제일 명확한 것 같은데 문제가 있습니다.

인프런에서 김영한님 기초 강의([링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8))를 보면 Configuration 파일 만드는 것을 선호하시고, 참고했던 블로그 글 작성하신 분은 @Qualifier를 별로 안좋아한다고 하십니다.

각각 그 이유가 김영한 님이 Configuration 파일을 따로 만들고 @Bean을 사용한 것은 변경 용이성 때문'강의 자료 내용: 여기서는 향후 메모리 리포지토리를 다른 리포지토리로 변경할 예정이므로, 컴포넌트 스캔 방식 대신에 자바 코드로 스프링 빈을 설정하겠다.'이고, 기술블로그 글 작성자 분이 @Qualifier를 별로 안좋아하는 이유는 '이름을 대체할 거라면 모르는데, 보통 같은 필드명을 두 번씩 중복하여 기재하는 것이 대부분이기 때문입니다.' 라고 적으셨습니다.

그런데 제 생각에는 @Qualifier 에 적힌 이름만 바꿔주면 되니까 @Component를 사용해도 변경 용이성에는 차이가 없는게 아닌가 생각되고, 변경 용이성을 고려해서 @Qualifier를 사용한다면 같은 필드명을 중복 기재한다고 해도 목적이 분명하니 괜찮지 않은가 생각이 됩니다. 그래서 다가올 팀 프로젝트에서는 @Component와 @Qualifier 조합으로 조금 더 사용해보고 어떤 문제점이 발생하는지 느껴보는게 좋겠습니다.
