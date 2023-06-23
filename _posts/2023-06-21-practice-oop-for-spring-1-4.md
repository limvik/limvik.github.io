---
layout: post
title: Spring을 위한 객체 지향 연습 1-4. 리플렉션 맛보기(1)
categories:
- OOP
tags:
- OOP
- Java
- Reflection
date: 2023-06-21 20:03 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/practice-oop-for-spring-1-3/)에서 리플렉션에 대해 공부해보기로 하고 살펴봤는데 처음엔 받아들이지 못하다가 실습하면서 조금이나 친숙해진 느낌입니다. 하지만 여전히 제대로 이해를 못했고, 단기간에 소화할만한 내용은 아닌 것 같아 이론적인 것은 적당히 넘어가고, 실습한 썰을 푸는 느낌으로 기록을 남겨보려 합니다.

## Reflective Programming

먼저 Reflective Programming에 대해 [Wikipedia](https://en.wikipedia.org/wiki/Reflective_programming)에서 역사적인 배경을 살펴봤습니다.

### Reflective Programming 의 역사적 배경

어셈블리어가 메인 프로그래밍 언어였던 시절에는 reflective programming 이라는 용어는 없었지만 구조 자체가 self-modifying code 를 사용하여 프로그래밍할 수 있었기 때문에 reflective한 특성을 가졌다고 합니다.

>self-modifying code Wikipedia 내용 GPT 요약 참고
>
>자기 수정 코드(self-modifying code, SMC)는 실행 중에 자신의 명령을 변경하는 코드로, 주로 명령 경로 길이를 줄이고 성능을 향상시키거나 단순히 반복적으로 유사한 코드를 줄이기 위해 사용됩니다. SMC는 `기존 명령을 덮어쓰거나 실행 시간에 새로운 코드를 생성하고 해당 코드로 제어를 전환하는 방식으로 구현`될 수 있습니다. 이 방법은 조건을 테스트해야 하는 횟수를 줄이기 위해 주로 사용되며, 초기화 중에만 또는 실행 중에 특정 프로그램 상태에 따라 수정을 수행할 수 있습니다. 자기 수정 코드는 어셈블리어와 고급 언어에서 사용될 수 있으며, 캐시 문제와 같은 고려해야 할 부작용이 있을 수 있습니다. 또한 자기 수정 코드는 보안 문제와 관련이 있을 수 있으며, 일부 환경에서는 사용할 수 없을 수도 있습니다.

그러다 C언어 같은 컴파일 언어의 인기로 잠시 묻혀있다가 1982년에 reflection 에 대한 논문이 발표됐다고 합니다. 객체 지향을 공부하다보면 항상 등장하는 smalltalk 에 reflection이 도입 됐었고, 이후 Java 에서는 1.1 버전에 reflection을 위한 API([Package java.lang.reflect](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html))가 등장합니다. 

### 간단한 Reflective Programming 예제

Java의 reflection API를 사용하는 것은 아니지만 간단하게 reflection의 맛(?)을 보는 예제를 살펴보겠습니다.

아래와 같이 클래스 이름을 출력하는 메서드를 가진 HelloWorld 클래스가 있습니다.

```java
public class HelloWorld {
    public void printClassName() {
        System.out.println("HelloWorld");
    }
}
```

이 클래스를 상속 받아, 상속 받은 클래스의 이름을 출력하기 위해 아래와 같이 작성할 수 있습니다.

```java
public class HelloLimvik extends HelloWorld {
    @Override
    public void printClassName() {
        System.out.println("HelloLimvik");
    }
}
```

그리고 간단하게 아래와 같이 출력해볼 수 있습니다.

```java
public class App {
    public static void main(String[] args) {
        new HelloWorld().printClassName();
        new HelloLimvik().printClassName();
    }
}
```

```
HelloWorld
HelloLimvik
```

하지만 이런 하드코딩 방식은 누가봐도 유연한 방식으로 보기는 어렵겠죠. 규모가 커져 많은 자식 클래스가 생긴다거나 하면 각각의 구현을 별도로 관리해줘야 합니다. 클래스 이름이 아니라 Fully Qualified Type Name 을 출력해야 한다면 여러모로 귀찮아 지겠죠.

`실행 시간(runtime)`에 호출되는 인스턴스에 따라 다른 클래스 이름이 출력되게 수정해보겠습니다.

```java
public class HelloWorld {
    public void printClassName() {
        System.out.println(this.getClass().getSimpleName());
    }
}
```

자식 클래스였던 HelloLimvik은 아래와 같이 Overriding을 할 필요가 없습니다.

```java
public class HelloLimvik extends HelloWorld {
    
}
```

그리고 실행하면 동일한 결과를 얻을 수 있습니다.

```
HelloWorld
HelloLimvik
```

책의 표현을 빌리자면, 리플렉션은 이처럼 실행 중인 프로그램이 자신과 소프트웨어 환경을 검사하고 발견한 내용에 따라 수행하는 작업을 변경하는 기능입니다.

> GPT-3.5를 이용한 추가적인 설명
>
> 주어진 예제 코드는 일부로 표현한 것이지만, 특정 클래스의 이름을 검사하고 해당 클래스의 정보에 따라 동작을 변경하는 작업을 수행합니다. 이런 관점에서, 주어진 코드는 리플렉션의 한 예라고 할 수 있습니다.
>
> 메소드 `printClassName()`은 `this.getClass().getSimpleName()`을 사용하여 현재 객체의 클래스 이름을 가져옵니다. 이것은 리플렉션의 한 형태로, 실행 중인 프로그램이 자신의 정보를 검사하는 것입니다. `getClass()` 메소드는 `Object` 클래스의 메소드로, 객체의 클래스 정보를 반환합니다. `getSimpleName()`은 `Class` 클래스의 메소드로, 클래스의 간단한 이름을 반환합니다.
>
> 따라서, `HelloLimvik` 클래스의 인스턴스를 만들고 `printClassName()` 메소드를 호출하면, "HelloLimvik"이라는 문자열이 출력됩니다. 이렇게 동적으로 클래스 이름을 가져와서 해당 정보에 따라 동작을 변경하는 것은 리플렉션의 한 예입니다.

처음에는 reflection 이 Java 만이 가진 기능인 줄 알았습니다. 알고보니 reflection 자체는 개념적인 것이고, 객체 지향 프로그래밍 언어의 클래스(Class)가 객체 지향 프로그래밍을 돕는 것처럼 Java의 reflection API는 Java에서 Reflective Programming을 할 수 있게 도와준다는 것을 알 수 있었습니다.

## Reflection API 사용해보기

이전에는 reflection API를 사용하지 않은 개념적인 것을 확인하기 위한 간단한 예제를 보았습니다. 이번에는 Reflection API를 사용한 간단한 계산기 예제를 살펴보겠습니다.

먼저 동적으로 불러올 Calculator 클래스와 add, multiply 메서드를 만들어 줍니다.

```java
public class Calculator {
    
    public int add(int a, int b) {
        return a + b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

}
```

그리고 실행시간에 사용자가 클래스 이름과 메서드 이름을 입력하여 동적으로 메서드를 불러와 사용하는 코드를 작성해줍니다.

```java
package com.limvik.calculator;

import java.lang.reflect.Method;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(System.in);

        System.out.print("클래스 이름을 입력하세요: ");
        String className = scanner.nextLine();

        System.out.print("메소드 이름을 입력하세요: ");
        String methodName = scanner.nextLine();

        // 클래스 이름으로 클래스 객체를 동적으로 가져옴
        Class<?> cls = Class.forName("com.limvik.calculator." + className);

        // 메소드 이름으로 메소드 객체를 동적으로 가져옴
        Method method = cls.getMethod(methodName, int.class, int.class);

        // 인스턴스 생성 (기본 생성자를 사용)
        Object instance = cls.getDeclaredConstructor().newInstance();

        // 메소드 호출
        System.out.println(method.invoke(instance, 5, 14));
    }
}
```

그리고 위 코드를 실행해보면, 실행 시간에 사용자가 입력한대로 클래스와 메서드가 선택되어 실행되는 것을 확인할 수 있습니다.

```
클래스 이름을 입력하세요: Calculator
메소드 이름을 입력하세요: add
19

클래스 이름을 입력하세요: Calculator
메소드 이름을 입력하세요: multiply
70
```

다시 또 책의 내용을 빌리자면 클래스 이름과 메서드를 가져오는 것(Class.forName(), cls.getMethod())과 같이 프로그램 스스로를 살펴보는 것을 `introspection` 이라 하고, 불러온 클래스의 메서드를 실행 시간에 호출하는 것(Java에서는 invoke 메서드 사용)을 `dynamic invocation` 이라고 합니다. 더 많은 reflection과 관련된 전문 용어나 이론적인 것이 있지만 먼저 공부해야할 우선순위 높은 것들이 많아서 간단하게만 봤습니다.

Oracle의 reflection 튜토리얼 중에 Retrieving Class Objects 섹션([링크](https://docs.oracle.com/javase/tutorial/reflect/class/classNew.html))을 보면 아래와 같은 문장으로 시작합니다.

> The `entry point` for all reflection operations is java.lang.Class.

그나마 reflection의 입구는 찾았으니 요번에는 아쉽지만 이정도에서 만족하고 Spring 흉내나 내보려고 합니다.

XML에 클래스를 등록하고, reflection을 이용해서 인스턴스를 만들어보겠습니다.

## XML에서 객체 불러오기

Spring을 배울 때 XML 에 bean을 등록하고, 사용하는 것부터 배웠는데, 사용하는 건 쉽지만 이해는 잘 안돼서 흉내라도 내보고 싶었습니다.

먼저 XML에 등록할 클래스 파일을 만들어 줍니다. XML에 등록하고 불러오는게 주 목적이니까 메서드는 생략합니다. 대신 뭔가 허전하니까 2개 만들어줍니다.

```java
// Seahorse.java
public class Seahorse {
    
}

// Plankton.java
public class Plankton {
    
}
```

그리고 이제 XML 파일을 만들고, 이전에 만들었던 클래스들을 등록합니다.

```xml
<!-- limviks.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<limviks>
    <limvik>com.limvik.xml.Seahorse</limvik>
    <limvik>com.limvik.xml.Plankton</limvik>
</limviks>
```

그리고 XML 파일에서 클래스 이름을 불러와서 인스턴스를 만드는 코드를 작성합니다.

```java
package com.limvik.xml;

import java.io.File;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

public class LoadObjectFromXml {
    public static void main(String[] args) {
        try {
            File xmlFile = new File("reflection/src/main/java/com/limvik/xml/limviks.xml");
    
            DocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();
            DocumentBuilder dBuilder = dbFactory.newDocumentBuilder();
            Document doc = dBuilder.parse(xmlFile);
    
            doc.getDocumentElement().normalize();
    
            NodeList nList = doc.getElementsByTagName("limvik");
    
            for (int temp = 0; temp < nList.getLength(); temp++) {
                Node nNode = nList.item(temp);
    
                if (nNode.getNodeType() == Node.ELEMENT_NODE) {
                    Element eElement = (Element) nNode;
                    String className = eElement.getTextContent();
    
                    Class<?> clazz = Class.forName(className);
                    Object obj = clazz.getDeclaredConstructor().newInstance();
    
                    System.out.println("Class Name : " + clazz.getSimpleName());
                    System.out.println("Object : " + obj.toString());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

XML 파일에서 클래스 이름을 가져오는 코드가 많아서 조금 길어보이지만, reflection 관련 코드는 짧습니다.

```java
Class<?> clazz = Class.forName(className);
Object obj = clazz.getDeclaredConstructor().newInstance();
    
System.out.println("Class Name : " + clazz.getSimpleName());
System.out.println("Object : " + obj.toString());
```

그리고 제 컴퓨터에서 출력한 결과는 아래와 같습니다.

```
Class Name : Seahorse
Object : com.limvik.xml.Seahorse@9807454
Class Name : Plankton
Object : com.limvik.xml.Plankton@1ddc4ec2
```

간단한 예제이긴 하지만, Spring에서 어떻게 XML에 등록된 객체를 불러왔는지 잘 이해가 안됐는데 조금이나마 궁금증이 해소된 느낌입니다. 물론 Spring은 훨씬 복잡한 코드를 가지고 있지만, XML에 등록된 객체를 읽고 reflection을 이용해서 인스턴스를 만들어주는 과정을 어느 정도 추상화해서 이해할 수는 있게 됐습니다.

그래도 뭔가 아쉬우니까 Spring 초기 버전이 제일 단순할 것 같아서 한 번 살펴보겠습니다.

## Spring Framework 0.9 살펴보기

XmlBeanFactory.java 파일에서 앞서 XML을 불러왔던 것과 비슷해 보이는 코드가 보입니다. 작은 일부분이지만 그래도 너무 길어서 다 붙여넣을까 말까 고민했지만, 그냥 넣습니다.

loadBeanDefinitions 메서드에서는 DocumentBuilderFactory, DocumentBuilder, Document 클래스를 이용해서 XML 파일을 불러오는 눈에 익은 코드가 보입니다. loadBeanDefinitions 메서드에서는 element를 불러와서 NodeList 타입에 저장하고 반복문 돌리는 것도 앞에서 봐서 익숙합니다. 그리고 loadBeanDefinition 메서드(단수형이라 s 빠짐)에서는 parseBeanDefinition 메서드를 호출하는데 여기서 클래스를 불러오는 코드가 보입니다. return new RootBeanDefinition(Class.forName(classname, true, cl), pvs, singleton);

```java
        /**
	 * Load definitions from this input stream and close it
	 */
	private void loadBeanDefinitions(InputStream is) throws BeansException {
		if (is == null)
			throw new BeanDefinitionStoreException("InputStream cannot be null: expected an XML file", null);

		try {
			logger.info("Loading XmlBeanFactory from InputStream [" + is + "]");
			DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
			logger.debug("Using JAXP implementation [" + factory + "]");
			factory.setValidating(true);
			DocumentBuilder db = factory.newDocumentBuilder();
			db.setErrorHandler(new BeansErrorHandler());
			db.setEntityResolver(new BeansDtdResolver());
			Document doc = db.parse(is);
			loadBeanDefinitions(doc);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException("ParserConfiguration exception parsing XML", ex);
		}
		catch (SAXException ex) {
			throw new BeanDefinitionStoreException("XML document is invalid", ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException("IOException parsing XML document", ex);
		}
		finally {
			try {
				if (is != null)
					is.close();
			}
			catch (IOException ex) {
				throw new FatalBeanException("IOException closing stream for XML document", ex);
			}
		}
	} // loadDefinitions (InputStream)

	/**
	 * Load bean definitions from the given DOM document.
	 * All calls go through this.
	 */
	private void loadBeanDefinitions(Document doc) throws BeansException {
		Element root = doc.getDocumentElement();
		logger.debug("Loading bean definitions");
		NodeList nl = root.getElementsByTagName(BEAN_ELEMENT);
		logger.debug("Found " + nl.getLength() + " <" + BEAN_ELEMENT + "> elements defining beans");
		for (int i = 0; i < nl.getLength(); i++) {
			Node n = nl.item(i);
			loadBeanDefinition((Element) n);
		}

		// Ask superclass to eagerly instantiate singletons
		preInstantiateSingletons();
	}

	/**
	 * Parse an element definition: wW know this is a BEAN element.
	 */
	private void loadBeanDefinition(Element el) throws BeansException {
		String id = getBeanId(el);
		logger.debug("Parsing bean definition with id '" + id + "'");

		// Create BeanDefinition now: we'll build up PropertyValues later
		AbstractBeanDefinition beanDefinition;

		PropertyValues pvs = getPropertyValueSubElements(el);
		beanDefinition = parseBeanDefinition(el, id, pvs);
		registerBeanDefinition(id, beanDefinition);

		String name = el.getAttribute(NAME_ATTRIBUTE);
		if (name != null && !"".equals(name)) {
			// Automatically create this alias. Used for
			// names that aren't legal in id attributes
			registerAlias(id, name);
		}
	}

	/**
	 * Parse a standard bean definition.
	 */
	private AbstractBeanDefinition parseBeanDefinition(Element el, String beanName, PropertyValues pvs) {
		String classname = null;
		boolean singleton = true;
		if (el.hasAttribute(SINGLETON_ATTRIBUTE)) {
			// Default is singleton
			// Can override by making non-singleton if desired
			singleton = TRUE_ATTRIBUTE_VALUE.equals(el.getAttribute(SINGLETON_ATTRIBUTE));
		}
		try {
			if (el.hasAttribute(CLASS_ATTRIBUTE))
				classname = el.getAttribute(CLASS_ATTRIBUTE);
			String parent = null;
			if (el.hasAttribute(PARENT_ATTRIBUTE))
				parent = el.getAttribute(PARENT_ATTRIBUTE);
			if (classname == null && parent == null)
				throw new FatalBeanException("No classname or parent in bean definition [" + beanName + "]", null);
			if (classname != null) {
				ClassLoader cl = Thread.currentThread().getContextClassLoader();
				return new RootBeanDefinition(Class.forName(classname, true, cl), pvs, singleton);
			}
			else {
				return new ChildBeanDefinition(parent, pvs, singleton);
			}
		}
		catch (ClassNotFoundException ex) {
			throw new FatalBeanException("Error creating bean with name [" + beanName + "]: class '" + classname + "' not found", ex);
		}
	}
```

간단한 수준에서 본 후에 봐서 그런지, 다른 로직이나 예외처리 등으로 코드는 복잡해졌지만, 흐름은 나름 이해할 수 있습니다. 아 뭔가 막힌게 조금이나마 해소된 느낌...

Spring Framework 0.9 버전은 sourceforge([링크](https://sourceforge.net/projects/springframework/files/springframework/))에서 받아보실 수 있습니다. XmlBeanFactory.java 파일은 src/com/interface21/beans/factory/xml 경로로 가시면 찾아보실 수 있습니다.

이번 글은 이정도로 마무리하고, 다음에는 Spring에서 Annotation을 사용하는 것을 따라해봐야겠습니다.

## Outro

처음에 reflection을 공부하기 시작했을 때 이걸 도대체 왜 쓰는건가 받아들이질 못해서 고생했습니다. Spring이 reflection을 사용해서 유연성과 확장성을 높여주는걸 몸소 느끼고 있음에도 뭔가 유연성과 확장성을 위해 많은걸 포기하는 느낌이 들었습니다. 아무래도 엔터프라이즈 레벨로 가면 유연성과 확장성 문제가 가장 큰 고민이기 때문이 아닐까 취준생이라 추측만 해봅니다.

Oracle의 reflection 튜토리얼 메인 페이지([링크](https://docs.oracle.com/javase/tutorial/reflect/))를 보면 단점이 꽤 큽니다. 안쓸 수 있으면 쓰지 말라고도 합니다.(DeepL 기계 번역입니다.)

>**리플렉션의 단점**
>
>리플렉션은 강력하지만 무분별하게 사용해서는 안 됩니다. 리플렉션을 사용하지 않고도 연산을 수행할 수 있는 경우에는 사용하지 않는 것이 좋습니다. 리플렉션을 통해 코드에 접근할 때는 다음과 같은 사항을 염두에 두어야 합니다.
>
>**성능 오버헤드**
>
>리플렉션에는 동적으로 확인되는 유형이 포함되므로 특정 Java 가상 머신 최적화를 수행할 수 없습니다. 따라서 리플렉션 연산은 비리플렉션 연산에 비해 성능이 느려지므로 성능에 민감한 애플리케이션에서 자주 호출되는 코드 섹션에서는 리플렉션을 사용하지 않도록 해야 합니다.
>
>**보안 제한**
>
>리플렉션을 사용하려면 런타임 권한이 필요한데, 보안 관리자 아래에서 실행할 때는 이 권한이 없을 수 있습니다. 이는 애플릿과 같이 제한된 보안 컨텍스트에서 실행해야 하는 코드의 경우 중요한 고려 사항입니다.
>
>**내부 노출**
>
>리플렉션을 사용하면 비공개 필드 및 메서드에 액세스하는 등 리플렉션이 없는 코드에서는 불법인 작업을 수행할 수 있으므로 리플렉션을 사용하면 예기치 않은 부작용이 발생하여 코드가 제대로 작동하지 않거나 이식성이 저하될 수 있습니다. 리플렉티브 코드는 추상화를 깨뜨리므로 플랫폼 업그레이드에 따라 동작이 변경될 수 있습니다.
>Translated with www.DeepL.com/Translator (free version)

아무리 트레이드오프라고는 하지만 객체 지향에서 캡슐화를 그렇게 외쳐대더니 유연성과 확장성을 위해 포기해버리니까 뭔가 이상하다 느낀 것도 있는 것 같습니다. 경험이 쌓이다 보면 둘 사이에서 잘 선택할 날이 오겠죠...?

마무리 글 쓰면서 뭔가 또 새로운 길로 빠져드는 느낌이들어 이만 마무리하겠습니다.
