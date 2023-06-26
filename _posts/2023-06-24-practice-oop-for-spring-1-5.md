---
layout: post
title: Spring을 위한 객체 지향 연습 1-5. 리플렉션 맛보기(2) - Custom Annotation 그리고 DI
categories:
- OOP
tags:
- OOP
- Java
- Reflection
- Spring
date: 2023-06-24 00:39 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/practice-oop-for-spring-1-4/)에서는 리플렉션을 이용해서 xml에 객체를 등록하고 불러오는 간단한 예제까지 해보고, Spring Framework 0.9 에서 xml을 이용하여 bean을 불러오는 코드를 살펴봤었습니다.

이번에는 언급했던 대로 Spring 흉내내기를 이어하면서 annotation을 사용하는 방법을 실습해보려고 합니다.

## 간단한 custom annotation 만들어보기

먼저 아래와 같은 문법으로 Seahorse 라는 custom annotation을 만듭니다.

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(value = RetentionPolicy.RUNTIME)
public @interface Seahorse {
    String value() default "Seahorse";
}
```

class, interface 등의 키워드가 들어가는 곳에 `@interface` 가 들어가 있는 것을 볼 수 있습니다. 그리고 value()는 필드를 선언하는 것 같지만 뒤에 ()가 있어 혼종같은 느낌입니다. 그리고 default 값을 지정해줄 수 있습니다.

`@Retention`과 같이 annotation에 사용하는 annotation을 meta annotation이라고 합니다. 그리고 @Retention annotation을 통해서는 custom annotation이 언제까지 유지되어야 하는지를 지정합니다. RetentionPolicy.`RUNTIME`은 컴파일 후에도 JVM에서 이 custom annotation을 읽을 수 있게 표시하는겁니다. 이렇게 하면 런타임에 reflection을 사용하여 이 custom annotation에 접근할 수 있습니다.

최대한 아무것도 안붙이고 만들어보려고 했는데, 런타임에 불러오기는 해야하니 어쩔 수 없습니다.

다음으론 Main 클래스에서 생성한 custom annotation을 붙여주면서 기본값이 아닌 "limvik"으로 값을 줍니다. 그리고 reflection을 이용해서 Main.class 로 Class 클래스를 불러오고 getAnnotation 메서드를 이용해서 사용했던 custom annotation 정보를 불러오고, 출력합니다.

```java
@Seahorse(value = "limvik")
public class Main {
    public static void main(String[] args) {
        Seahorse seahorse = Main.class.getAnnotation(Seahorse.class);
        System.out.println(seahorse.value());
    }
}
```

그러면 아래와 같이 Main 클래스에서 custom annotation인 Seahorse 선언 시 value 값으로 지정했던 limvik이 출력되는 것을 볼 수 있습니다.

```
limvik
```

### Spring Annotation 살펴보기

이 예제로 실습해보고 나니까, Spring에서 `@Qualifier` 를 이용하던 것이 생각이 났습니다.

그래서 @Qualifier annotation 코드를 확인해보니 아래와 같았습니다.

```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * This annotation may be used on a field or parameter as a qualifier for
 * candidate beans when autowiring. It may also be used to annotate other
 * custom annotations that can then in turn be used as qualifiers.
 *
 * @author Mark Fisher
 * @author Juergen Hoeller
 * @since 2.5
 * @see Autowired
 */
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {

	String value() default "";

}
```

@Target은 딱히 찾아보지 않아도 annotation 사용 대상을 지정한다는 것을 파악할 수 있습니다. 그리고 정체모를 annotation도 있지만, annotation을 지금 깊게 파볼 생각은 아니라서 스킵하고 넘어가겠습니다.

정체모를 meta annotations를 제외하고 보면, 앞서 실습한 custom annotation과 비슷하게 파악할 수 있습니다. Spring 에서 런타임에 reflection을 이용해서 annotation에 설정된 값을 가져갈 것이라고 추측 정도는 해볼 수 있겠습니다.

근데 또 이게 궁금해져가지고... @Qualifier로 지정한 대상에 어떻게 DI(Dependency Injection)까지 되는지 궁금해져서 몇 시간동안 헤메이다가 모르는게 너무 많아서 지금 당장 다룰 수 있는게 아닌 것 같아 중단했습니다. Github에 있는 Spring 코드를 살펴보기도 하고 GPT에 물어보기도 했는데, 별 도움은 안되겠지만 관련 내용들은 글 마지막에 붙여넣도록 하겠습니다.

아쉬운대로 간단하게 Custom Annotation을 이용해서 DI Container를 통해 DI 하는 예제를 만들어보겠습니다.

## 간단한 Custom Annotation을 이용해 DI 해보기

### Field Injection

간단하게 먼저 Field Injection 을 위해서 아래와 같이 Custom Annotation을 만들어줍니다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(value = RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Inject {
    
}
```

@Qualifier 에서 봤던 @Target을 사용해서 이번에는 Field로 한정했습니다.

다음으로 Limvik 클래스와 의존성 주입 대상이 될 Seahorse 클래스를 만들어줍니다.

```java
public class Limvik {
    @Inject()
    Seahorse seahorse;
}

public class Seahorse {
    public void swim() {
        System.out.println(this.getClass().getSimpleName() + " is swimming!");
    }
}
```

swim 메서드에 reflection 쓸 이유가 전혀 없지만, reflection 공부한 김에 써줬습니다.

다음으로 DI 대상이 1개라 Container 까지는 필요가 없지만, 아쉬움에 간단한 DI Container 예제도 찾아서 만들어 줍니다.

```java
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;

public class DIContainer {
    
    private Map<Class<?>, Object> instances = new HashMap<>();

    <T> T getInstance(Class<T> clazz) throws Exception {
        if (!instances.containsKey(clazz)) {
            T instance = createInstance(clazz);
            instances.put(clazz, instance);
        }
        return (T) instances.get(clazz);
    }

    private <T> T createInstance(Class<T> clazz) throws Exception {
        T instance = clazz.getDeclaredConstructor().newInstance();
        for (Field field : clazz.getDeclaredFields()) {
            if (field.isAnnotationPresent(Inject.class)) {
                Object dependency = getInstance(field.getType());
                boolean wasAccessible = field.canAccess(instance);
                field.setAccessible(true);
                field.set(instance, dependency);
                field.setAccessible(wasAccessible);
            }
        }
        return instance;
    }
}
```

createInstance() 메서드부터 보면 getDeclaredConstructor() 메서드로 생성자를 불러와서 newInstance() 메서드로 새로운 인스턴스(instance)를 만들고, Field에 @Inject 로 annotation 된 것이 있으면 field의 타입을 확인한 후 해당 타입의 인스턴스(dependency)를 만듭니다. 그리고 Field 에 의존성을 주입합니다. 여기서 만들어진 인스턴스들은 모두 instances 에 저장되어 하나의 인스턴스만 유지됩니다. Spring으로 따지자면 beans라고 할 수 있습니다. Spring은 singleton 방식 외에도 인스턴스를 관리하는 방식이 여러가지가 있기는 합니다. 하지만 여기선 단순하게...

장난 삼아 limviks로 하고 싶었는데, 혼란스러울까봐 참았습니다.

다음으로 DI Container 인스턴스를 만들고 실행해 봅니다.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        DIContainer diContainer = new DIContainer();
        diContainer.getInstance(Limvik.class).seahorse.swim();
    } 
}
```

위 코드를 실행하면 아래와 같은 출력을 볼 수 있습니다.

```
Seahorse is swimming!
```

Component Scanning 도 흉내내 보고 싶기는 한데... 지금 취준생으로서 우선순위는 아닌 것 같아 아쉽지만 미뤄둡니다.

그보다 Field에 의존성을 주입하는 코드를 보면 Field Injection을 왜 권장하지 않는지 알 수 있습니다.

```java
Object dependency = getInstance(field.getType());
boolean wasAccessible = field.canAccess(instance);
field.setAccessible(true);
field.set(instance, dependency);
field.setAccessible(wasAccessible);
```

강제로 접근 가능하게 만들어 set을 해버립니다. 객체 지향 패러다임에서는 캡슐화를 깨부수는 몹쓸 짓(?)이죠.

심지어 이렇게 private으로 선언해도 가능합니다.

```java
public class Limvik {

    @Inject
    private Seahorse seahorse;

    public void callSeahorse() {
        seahorse.swim();
    }
    
}

public class Main {
    public static void main(String[] args) throws Exception {
        DIContainer diContainer = new DIContainer();
        diContainer.getInstance(Limvik.class).callSeahorse(); // Seahorse is swimming!
    } 
}
```

그런데 이러면... 권장되고 주로 사용되는 Constructor Injection이 궁금해질 수 밖에 없습니다.

### Constructor Injection

이번에는 @Inject custom annotation의 target을 constructor로 지정해줍니다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(value = RetentionPolicy.RUNTIME)
@Target(ElementType.CONSTRUCTOR)
public @interface Inject {
    
}
```

그리고 이번 DI Container는 Constructor에 의존성을 주입할 수 있도록 수정합니다.

```java
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;

public class DIContainer {
    private Map<Class<?>, Object> instances = new HashMap<>();

    <T> T getInstance(Class<T> clazz) throws Exception {
        if (!instances.containsKey(clazz)) {
            T instance = createInstance(clazz);
            instances.put(clazz, instance);
        }
        return (T) instances.get(clazz);
    }

    private <T> T createInstance(Class<T> clazz) throws Exception {
        Constructor<T> constructor = getInjectConstructor(clazz);
        if (constructor == null) {
            return clazz.getDeclaredConstructor().newInstance();
        } else {
            return constructor.newInstance(getDependencies(constructor));
        }
    }

    private <T> Constructor<T> getInjectConstructor(Class<T> clazz) {
        for (Constructor<?> constructor : clazz.getDeclaredConstructors()) {
            if (constructor.isAnnotationPresent(Inject.class)) {
                return (Constructor<T>) constructor;
            }
        }
        return null;
    }

    private Object[] getDependencies(Constructor<?> constructor) throws Exception {
        Class<?>[] paramTypes = constructor.getParameterTypes();
        Object[] dependencies = new Object[paramTypes.length];
        for (int i = 0; i < paramTypes.length; i++) {
            dependencies[i] = getInstance(paramTypes[i]);
        }
        return dependencies;
    }
}
```

예제를 제공해준 GPT-4 에 감사...

이번에는 이전 Field 클래스의 set() 메서드 처럼 직관적이지가 않습니다. Constructor의 newInstance() 메서드는 공식 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/reflect/Constructor.html#newInstance%28java.lang.Object...%29))를 보면, public T newInstance(Object... initargs) 와 같이 초기화할 객체들을 0개 이상 가변적으로 받을 수 있습니다. 그래서 이번 예제의 createInstance() 메서드는 annotation 된 constructor를 찾지 못한 경우 default constructor 를 호출하여 인스턴스를 생성하고, annotation 된 constructor가 있는 경우 해당 constructor의 parameter 를 찾아 인스턴스 배열을 만들고, Constructor의 newInstance() 메서드 인수로 넘깁니다.

Field 에 강제로 접근 가능하게 만들고 의존성을 주입하는 것 보다 더 객체 지향적이라 판단됩니다.

그리고 이번에는 Constructor에 여러 개의 파라미터가 있어도 잘 들어가는지 보기 위해 Injection 할 클래스를 추가로 만들어줍니다.

```java
public class Seahorse {
    public void chasePlankton() {
        System.out.println("Chasing the plankton");
    }
}

public class Plankton {
    public void escapeSeahorse() {
        System.out.println("Escape from the seahorse");
    }
}

public class Limvik {
    private Seahorse seahorse;
    private Plankton plankton;

    @Inject
    public Limvik(Seahorse seahorse, Plankton plankton) {
        this.seahorse = seahorse;
        this.plankton = plankton;
    }

    public void play() {
        seahorse.chasePlankton();
        plankton.escapeSeahorse();
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        DIContainer diContainer = new DIContainer();
        diContainer.getInstance(Limvik.class).play();
    }
}

```

그리고 실행해보면 이상 없이 의존성이 잘 주입된 것을 확인할 수 있습니다.

```
Chasing the plankton
Escape from the seahorse
```

## Spring Framework 6.0.x 코드 훑어보기

[ComponentScanBeanDefinitionParser](https://github.com/spring-projects/spring-framework/blob/main/spring-context/src/main/java/org/springframework/context/annotation/ComponentScanBeanDefinitionParser.java#L268), [ReflectionUtils](https://github.com/spring-projects/spring-framework/blob/35667e81ea24c83f363092ee36d01ac32d2e6bf4/spring-core/src/main/java/org/springframework/util/ReflectionUtils.java#L182) 에서 아래 같은 코드를 보면, 어딘가에서 만들었던 예제와 비슷한 일을 할 것 같기는 한데, 찾기가 쉽지 않습니다.

```java
// ComponentScanBeanDefinitionParser
// ...
result = ReflectionUtils.accessibleConstructor(ClassUtils.forName(className, classLoader)).newInstance();

// ReflectionUtils
/**
* Obtain an accessible constructor for the given class and parameters.
* @param clazz the clazz to check
* @param parameterTypes the parameter types of the desired constructor
* @return the constructor reference
* @throws NoSuchMethodException if no such constructor exists
* @since 5.0
*/
public static <T> Constructor<T> accessibleConstructor(Class<T> clazz, Class<?>... parameterTypes) throws NoSuchMethodException {

    Constructor<T> ctor = clazz.getDeclaredConstructor(parameterTypes);
	makeAccessible(ctor);
	return ctor;
}
```

더 찾아보고 싶지만, 더 공부한 후에 지식이 쌓이고 찾는게 시간도 아끼고 좋을 것 같아 여기까지 합니다.

## Outro

파면 팔수록 새로운게 더 많이 튀어나와서 이번에도 머릿속에 대혼란이 찾아왔습니다. 아직도 혼란스럽기는 하지만 계속 공부하면서 정리해 나가야겠습니다.

Spring 에서 원하는 코드를 찾아내진 못했지만, 너무 멀게만 느껴졌던 reflection과 annotation을 이용한 DI가 조금이나마 가까워진 기분입니다. 디버거 사용해서 코드를 조사하는 방법을 연습하고 있는데, 이 방법으로 원하는 코드를 찾을 수 있을지 확인해 봐야겠습니다.

그보다 이제는 취업용 포트폴리오 준비를 해야하니, 다음 Spring 관문인 AOP까지 해보는데는 시간이 좀 걸리겠습니다.


## Spring Framework 6.0.x DI 관련 코드 찾아본 기록

annotation이 Spring 0.9에서는 없었기 때문에 Github에서 [AutowireCandidateQualifier.java](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/support/AutowireCandidateQualifier.java), [QualifierAnnotationAutowireCandidateResolver.java](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/annotation/QualifierAnnotationAutowireCandidateResolver.java#L226), [AttributeAccessorSupport.java](https://github.com/spring-projects/spring-framework/blob/06b6c4bcf902636d16e6d151c74e9cc536e2d6ec/spring-core/src/main/java/org/springframework/core/AttributeAccessorSupport.java#L59), [QualifierAnnotationAutowireContextTests.java](https://github.com/spring-projects/spring-framework/blob/main/spring-context/src/test/java/org/springframework/beans/factory/support/QualifierAnnotationAutowireContextTests.java#L114) 외에도 여러가지 Qualifier 단어가 들어간 클래스나 부모 클래스 그리고 테스트 코드들을 살펴봤습니다. 살펴보다 보니 뭔가 한 군데 모으고는 있나보다 싶기는 한데, Scanning은 어떻게 한건지 그리고 어떻게 DI container 까지 전달 돼서 DI가 어떻게 되는건지 아직 제 지식으로는 추적이 안됩니다.

그래서 역으로 타고 가보자! 했는데, 어디부터 시작해야될지 몰라서 GPT-4에 물어봤습니다.

> Spring Framework에서 의존성 주입이 이루어지는 대부분의 로직은
> `org.springframework.beans.factory.support` 패키지의
> `DefaultListableBeanFactory` 클래스에서 찾을 수 있습니다. 이 클래스는 Spring의 핵심
> IoC(Inversion of Control) 컨테이너의 기본 구현체입니다.
> 
> 이 클래스 내부에서는 빈의 생성, 설정, 초기화, 소멸 등의 라이프사이클을 관리하며, 그 과정에서 의존성 주입이 이루어집니다.
> 
> `@Autowired`와 `@Qualifier` 어노테이션을 통한 의존성 주입은
> `AutowiredAnnotationBeanPostProcessor` 클래스에서 처리됩니다. 이 클래스는
> BeanPostProcessor 인터페이스를 구현하는데, 이 인터페이스의 구현체는 스프링 컨테이너가 빈을 생성한 후에 초기화를
> 수행하기 전과 후에 콜백 메서드를 호출하여 빈의 추가적인 처리를 수행할 수 있게 합니다.
> 
> 이때 `AutowiredAnnotationBeanPostProcessor`는 초기화 전에 호출되는
> `postProcessProperties` 메서드를 통해 `@Autowired`와 `@Qualifier` 어노테이션을
> 처리합니다. 이 메서드 내부에서는 `AutowiredAnnotationBeanPostProcessor#inject` 메서드를
> 호출하여 의존성 주입을 수행합니다.
> 
> 따라서 의존성 주입 로직을 좀 더 자세히 알고 싶다면 `DefaultListableBeanFactory` 클래스와
> `AutowiredAnnotationBeanPostProcessor` 클래스를 살펴보시면 좋을 것 같습니다.

[DefaultListableBeanFactory](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultListableBeanFactory.java), [AutowiredAnnotationBeanPostProcessor](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.java) 부터 시작해서 살펴봤지만, 여전히 이해가 안됩니다.

또 단서를 찾아볼겸 GPT-4에 요청을 청해봅니다.

> 네, 알겠습니다. Spring의 의존성 주입 과정은 상당히 복잡합니다. 아래는 `@Autowired`와 `@Qualifier`를 사용하는 과정에서 거치는 주요 단계입니다. 이를 통해 Spring이 어떻게 의존성 주입을 처리하는지 간략하게 이해할 수 있을 것입니다.
> 
> 1. `org.springframework.context.support.AbstractApplicationContext` 클래스의 `refresh` 메서드: 스프링 어플리케이션 컨텍스트의 초기화 및 리프레시(재실행) 과정을 담당합니다. 이 과정에서 빈 팩토리의 설정, 빈의 생성 및 등록, 빈 포스트 프로세서의 등록 등이 이루어집니다.
> 
> 2. `org.springframework.beans.factory.support.DefaultListableBeanFactory` 클래스의 `preInstantiateSingletons` 메서드: 등록된 빈 정의를 바탕으로 빈을 생성하고, 이를 빈 팩토리의 싱글턴 맵에 등록하는 과정을 수행합니다.
> 
> 3. `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory` 클래스의 `initializeBean` 메서드: 빈의 초기화 과정을 담당합니다. 이 과정에서 빈 포스트 프로세서의 `postProcessBeforeInitialization` 메서드와 `postProcessAfterInitialization` 메서드가 호출됩니다.
> 
> 4. `org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor` 클래스의 `postProcessProperties` 메서드: 빈의 프로퍼티에 대한 후처리를 담당합니다. 이 메서드에서 `@Autowired`와 `@Qualifier` 어노테이션을 처리하며, 실제로 의존성 주입이 이루어집니다. 이 메서드는 빈의 초기화 전에 호출되므로, 이 시점에서 의존성이 주입됩니다.
> 
> 5. `org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor` 클래스의 `inject` 메서드: 실제로 의존성 주입을 수행하는 메서드입니다. 이 메서드 내부에서는 `@Autowired` 어노테이션이 표시된 필드나 메서드에 대해 주입 대상이 되는 빈을 조회하고, 이를 대상 객체에 주입합니다.
> 
> `@Qualifier` 어노테이션은 `inject` 메서드 내부에서 `@Autowired` 어노테이션을 처리하는 과정 중에 함께 처리됩니다. `@Qualifier`가 지정된 경우, `@Autowired`로 주입할 빈을 선택하는 기준에 추가로 사용됩니다. 그래서 `@Qualifier` 어노테이션을 통해 같은 타입의 여러 빈 중에서 원하는 빈을 정확하게 선택할 수 있습니다.
> 
> 이러한 과정을 거쳐서 `@Autowired`와 `@Qualifier` 어노테이션을 사용하여 의존성 주입이 이루어집니다. 이것이 Spring의 주요 의존성 주입 과정입니다. 다만, Spring은 매우 크고 복잡한 프레임워크이므로, 이외에도 더 많은 세부 과정이 있습니다. 그러나 위의 과정을 통해 Spring이 의존성 주입을 어떻게 처리하는지 대략적으로 알 수 있습니다.

보다가 이해가 안돼고 답답해서 Spring 관련 책도 찾아보니, 이 과정을 이해하려면 잠깐 공부해서 될만한게 아니라는 사실을 깨닫고 훗날 복수(?)를 다짐하며 중단했습니다.
