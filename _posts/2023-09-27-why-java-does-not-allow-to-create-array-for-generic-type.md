---
layout: post
title: Java는 왜 Generic Type 배열 생성을 허용하지 않을까?
categories:
- Java
- Generics
tags:
- Java
- Generics
date: 2023-09-27 20:00 +0900
---
## Intro

코딩테스트 문제 풀면서([링크](/posts/programmers-table-merge-150366/)) Generic 연습도 할겸, Generic을 사용했었습니다.

아래와 같이 Generic Type인 T의 2차원 배열인 data를 Table 클래스 생성자에서 인스턴스를 만들지 않고, 외부에서 인스턴스를 받아오도록 작성했습니다.

```java
class Table<T> {
        
    private final T[][] data;
    private final UnionFind unionFind;
    
    public Table(T[][] data, UnionFind unionFind) {
        this.data = data;
        this.unionFind = unionFind;
    }
    // ... 
}
```

이렇게 작성하기 전에는 Generic에 대한 지식이 부족해서, 생성자에서 배열 크기(size)를 받아 인스턴스 생성하는 것을 시도했습니다.

```java
// UnionFind는 생략
class Table<T> {
    private final T[][] data;

    public Table(int size) {
        this.data = new T[size][size];
    }
}
```

그러면 아래와 같이 generic array creation 메시지와 함께 컴파일 타임 error가 발생합니다.

```bash
/Main.java:15: error: generic array creation
        this.data = new T[size][size];
                    ^
1 error
```

문제 푸는데만 집중해서 이 사실을 잊고 있었는데, Coursera 에서 Algorithms, part1 강의를 듣는 중 이에대한 해결 방법이 나왔습니다.

아래와 같이 형 변환을 해주는 것입니다.

```java
class Table<T> {
    private final T[][] data;

    public Table(int size) {
        this.data = (T[][]) new Object[size][size];
    }
}
```

그런데 강의하시는 교수님은 `개발자의 실수`를 유발할 수 있어 형 변환을 하는 코드가 좋은 코드는 아니라 생각하시는데, Java의 Generic 구현의 한계 상 사용할 수 밖에 없다고 불만을 표시하십니다. 이렇게 형 변환을 하면 unchecked cast 메시지와 함께 warning이 발생하는데, 여기에 사과 메시지를 넣어야한다고 농담도 하십니다.

왜 안되는지에 대해서는 강의 주제를 벗어나기 때문에 그냥 넘어가셔서, 궁금해서 찾아봤습니다.

Generic에 대한 일반적인 이야기부터 살펴보겠습니다.

## 왜 Generics를 사용할까?

[https://docs.oracle.com/javase/tutorial/java/generics/index.html](https://docs.oracle.com/javase/tutorial/java/generics/index.html)
[https://docs.oracle.com/javase/tutorial/extra/generics/intro.html](https://docs.oracle.com/javase/tutorial/extra/generics/intro.html)

Oracle의 java 문서를 보면 Generic은 `컴파일 타임`에 type의 정확성을 체크하여 버그를 조기에 발견할 수 있는 기능입니다.

하지만 이러한 목적만을 위해 Generic을 사용하지는 않습니다. 앞서 Table 클래스를 만들 때 확장성을 생각하며 다양한 타입의 데이터를 Table 클래스에서 다루기 위해 Generic을 사용했습니다.

Oracle 문서의 `Generics를 왜 사용`하는지에 대한 섹션([Why Use Generics?](https://docs.oracle.com/javase/tutorial/java/generics/why.html))에서는 장점 3 가지를 언급하고 있습니다.

1. 컴파일 타임에 강력한 타입 체크
2. 형 변환 제거
3. 개발자에게 Generic Algorithms 구현을 허용

### 컴파일 타임에 강력한 타입 체크

Generic을 사용하면 컴파일 타임에 체크를 함으로써, 런타임에 발생할 수 있는 버그를 조기에 발견할 수 있습니다.

Java 기본서를 통해 볼 수 있는 내용입니다.

### 형 변환 제거

형 변환이 개발자의 실수를 유발할 수 있고, 아래와 같이 Collections 에서 Generic이 없으면 매번 형 변환을 해줘야하는 불편함이 있으므로, 제거 대상이 되어야 한다는 것을 고려했을 때 납득이 되는 장점입니다. 여러 타입이 들어가 있을때는 타입에 맞춰 형 변환 해줘야하는걸 생각해보면 끔찍하기도 합니다.

```java
// The following code snippet without generics requires casting:
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);

// When re-written to use generics, the code does not require casting:
List<String> list = new ArrayList<String>();
list.add("hello");
String s = list.get(0);   // no cast
```

### 개발자에게 Generic Algorithms 구현을 허용

개인적으로는 Generic 이라는 이름에 가장 어울리는 사용목적이라 생각합니다. 타입을 추상화 (혹은 일반화)해서 코드 중복을 방지해줍니다.

```java
class Table<T> {
    private final T[][] data;

    public Table(int size) {
        this.data = (T[][]) new Object[size][size];
    }
}
```

만약 Generic 이 없었다면, Table 클래스는 다른 타입의 데이터를 다루기 위해 타입별로 다른 Table 클래스를 만들어야 합니다. 다른 방법을 생각해보자면 필드에는 Object 타입으로 선언 후, 무슨 타입을 사용할지도 입력으로 받아, 형 변환해서 사용하는 방법이 있겠습니다.

일반적인 Java 사례를 생각해보면, Stack, ArrayList 도 generic을 이용해서 타입을 제한하기도 하지만, 하나의 클래스로 다양한 데이터 타입에 대응할 수 있게 합니다.

#### Generic Algorithms의 정의

다른 분들이 정의한 Generic algorithms의 정의를 살펴보면, Generic의 정의라고 해도 괜찮아 보입니다.

>Generic algorithms are those that can work with different types of data or inputs, without requiring any modification. For example, a generic sorting algorithm can sort any collection of items that can be compared, such as numbers, strings, or custom objects.  
>
>번역) Generic algorithms는 변경 없이 다른 데이터 타입이나 입력으로 작동할 수 있는 알고리즘입니다. 예를 들어, generic sorting algorithm은 숫자, 문자열, 또는 사용자 지정 객체와 같이 비교할 수 있는 모든 항목의 collection을 정렬할 수 있습니다.

출처: [How do you choose between generic and specific algorithms for your projects?](https://www.linkedin.com/advice/1/how-do-you-choose-between-generic-specific)  

---

개인적인 Generics의 키워드를 뽑아보자면, 타입 안정성(type safety), 재사용성(reusability) 정도가 되겠습니다.

하지만 일반화시키는 만큼 특정 요구사항에 최적화된 알고리즘을 만들기는 어렵다는 점이 단점으로 언급되고 있습니다.

---

다음으로 제가 겪은 문제는 Generic Type 배열이 생성되지 않는 문제였지만, 조금 더 간단하게 살펴보기 위해 Generic Type 인스턴스를 생성할 수 없는 문제를 살펴보겠습니다.

## Java는 왜 Generic Type 인스턴스 생성을 허용하지 않을까?

Generics 제한(restriction)에 대한 문서 내용([링크](https://docs.oracle.com/javase/tutorial/java/generics/restrictions.html#createObjects))에서도 new 키워드를 이용해서 인스턴스를 생성하는 것은 불가능하다는 것을 보여주고 있습니다.

### Cannot Create Instances of Type Parameters

```java
// You cannot create an instance of a type parameter. For example, the following code causes a compile-time error:

public static <E> void append(List<E> list) {
    E elem = new E();  // compile-time error
    list.add(elem);
}

// As a workaround, you can create an object of a type parameter through reflection:

public static <E> void append(List<E> list, Class<E> cls) throws Exception {
    E elem = cls.newInstance();   // OK
    list.add(elem);
}

// You can invoke the  append  method as follows:

List<String> ls = new ArrayList<>();
append(ls, String.class);
```

문서 예제에서는 reflection을 사용해서 인스턴스를 만들고 있습니다. 새로운 배열 인스턴스를 만든다면 reflect 패키지의 [Array](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Array.html) 클래스를 사용할 수 있습니다.

그냥 이것만 보면 런타임에도 타입 체크해서 타입 지정하고, 새로운 인스턴스 만들면 되는거 아닌가? 라는 생각이 듭니다. 그런데 왜 안되는지 결론부터 말씀 드리자면, `런타임`에는 Generic Type 정보가 `삭제`되어있기 때문입니다.

공식적인 용어로는 `Type Erasure`라 합니다.

### Type Erasure

문서([링크](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html))에 나와있는 Type Erasure에 대한 소개부터 살펴보겠습니다.

>Generics were introduced to the Java language to provide `tighter type checks at compile time` and to `support generic programming`. To implement generics, the Java compiler applies `type erasure` to:
>
>-   `Replace all type parameters` in generic types with their bounds or  `Object`  if the type parameters are unbounded. The produced bytecode, therefore, contains only ordinary classes, interfaces, and methods.  
>-   `Insert type casts` if necessary to preserve type safety.  
>-   `Generate bridge methods` to preserve polymorphism in extended generic types.
>
>Type erasure ensures that `no new classes are created for parameterized types`; consequently, generics incur no runtime overhead`.

내용을 번역해 보자면 아래와 같습니다.

>컴파일 타임에 타입 체크를 더 빡세게 하고, generic programming을 지원하도록 Java에 Generics를 도입하였습니다.
>
>Generics를 구현하기 위해 Java 컴파일러가 적용하는 `type erasure` :
>
>- type parameters에 bounds가 있다면 해당 bounds로, 없다면 Object 로 type parameters를 모두 교체하여 생성된 바이트코드에는 일반적인 클래스, 인터페이스 및 메서드만 포함됩니다.
>- type safety를 유지하기 위해 필요하다면 형 변환 삽입  
>- extended generic types에서 다형성을 유지하기 위해 bridge methods 생성  
>
>type erasure는 parameterized types에 대해 새 클래스가 생성되지 않도록 보장하므로 generics는 런타임 오버헤드가 발생하지 않습니다.

하나씩 살펴보겠습니다.

#### type parameters를 모두 교체

type parameters를 교체하기는 하는데 조건에 따라 출력이 달라집니다. bounds가 있다면 해당 bounds로 없다면 Object로 교체합니다.

bounds 라는 단어는 알지만, 정확히 무엇을 의미하는건지 잘 와닿지 않습니다. 느낌적인 느낌으로 꺽쇠 괄호<>에 지정한 type이 bounds를 의미하는 것 같기는 합니다.

하지만 문서([bounded type parameter](https://docs.oracle.com/javase/tutorial/java/generics/bounded.html))의 내용에 따라 구체적으로 보자면, 저의 느낌적인 느낌은 틀립니다.

아래 예제 코드를 통해 보면 \<T> 는 bounds가 `없`고, \<U extends Number> 는 bounds가 `있`습니다.

```java
public class Box<T> {

    private T t;          

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }

    public <U extends Number> void inspect(U u){
        System.out.println("T: " + t.getClass().getName());
        System.out.println("U: " + u.getClass().getName());
    }

    public static void main(String[] args) {
        Box<Integer> integerBox = new Box<Integer>();
        integerBox.set(new Integer(10));
        integerBox.inspect("some text"); // error: this is still String!
    }
}
```

`T`만 Integer로 지정했음에도 불구하고, upper bound를 Number로 지정한 `U` 에 String을 대입해서 에러가 발생하는 예제입니다.

이때 `extends` 는 upper bound를 설정하기 위한 키워드로 인터페이스를 지정할 때도 extends를 사용합니다.

예를들어 Iterable 인터페이스로 upper bound를 설정한다면 아래와 같이 extends 키워드와 함께 사용해야합니다.

```java
class Table<T extends Iterable> {
    private final T[][] data;

    public Table(int size) {
        this.data = (T[][]) new Object[size][size];
    }
}
```

꺽쇠 괄호 내에 extends 키워드를 사용해야 upper bound를 설정했다고 할 수 있습니다.

`super` 키워드로 lower bound를 지정하는 내용의 문서([Lower Bounded Wildcards](https://docs.oracle.com/javase/tutorial/java/generics/lowerBounded.html))도 있지만, Type Erasure에서 언급하고 있는 내용은 upper bound를 이야기하고 있으므로 다음에 Generics의 Wildcards를 다룰 기회가 있을 때 따로 다뤄보겠습니다.

그럼 이제 bound가 뭘 의미하는지 알았으니, 교체되는 문서([Erasure of Generic Types](https://docs.oracle.com/javase/tutorial/java/generics/genTypes.html)) 예제를 살펴보겠습니다.

##### unbounded type 교체

extends 키워드를 사용하지 않은 unbounded type 예제입니다.

```java
public class Node<T> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}
```

\<T>는 사라지고, T는 bound가 없으므로 Java Compiler에 의해 Object로 교체됩니다.

```java
public class Node {

    private Object data;
    private Node next;

    public Node(Object data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Object getData() { return data; }
    // ...
}
```

##### bounded type 교체

다음은 extends 키워드를 사용한 bounded type 예제입니다.

```java
public class Node<T extends Comparable<T>> {

    private T data;
    private Node<T> next;

    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }

    public T getData() { return data; }
    // ...
}
```

\<T>는 제거되고, T는 upper bound로 설정한 Comparable로 교체됩니다.

```java
public class Node {

    private Comparable data;
    private Node next;

    public Node(Comparable data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Comparable getData() { return data; }
    // ...
}
```

##### Generic Method에서의 교체

Generic Method인 경우에도 동일하게 교체됩니다.

문서([Erasure of Generic Methods](https://docs.oracle.com/javase/tutorial/java/generics/genMethods.html))의 예제를 보면 쉽게 파악할 수 있습니다.

```java
// The Java compiler also erases type parameters in generic method arguments. 
// Consider the following generic method:

// Counts the number of occurrences of elem in anArray.
//
public static <T> int count(T[] anArray, T elem) {
    int cnt = 0;
    for (T e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}
// Because T is unbounded, the Java compiler replaces it with Object:

public static int count(Object[] anArray, Object elem) {
    int cnt = 0;
    for (Object e : anArray)
        if (e.equals(elem))
            ++cnt;
        return cnt;
}
// Suppose the following classes are defined:

class Shape { /* ... */ }
class Circle extends Shape { /* ... */ }
class Rectangle extends Shape { /* ... */ }
// You can write a generic method to draw different shapes:

public static <T extends Shape> void draw(T shape) { /* ... */ }
// The Java compiler replaces T with Shape:

public static void draw(Shape shape) { /* ... */ }
```

#### Object 인스턴스를 생성하면 되는거 아닌가?

라고 생각하기에는, Object 인스턴스를 만들면 모든 타입을 허용하게 돼서 Generic을 사용하는 의미가 없어지게 됩니다. bounds를 설정하더라도 인터페이스인 경우에는 new 키워드를 사용해서 인스턴스를 생성할 수 없는 문제가 있습니다. 여러가지로 문제가 많이 발생하는 것을 알 수 있습니다.

그럼 다시 원래 문제였던 배열로 가본다면,

#### Object 배열을 만들면 되는거 아닌가?

라는 생각도 들 수 있습니다. 그런데 Object 배열을 만들게되면, 인스턴스 생성과 비슷하게 타입 문제가 발생합니다.

아래 코드는 new T\[size]\[size]; 할 때와 마찬가지로 error가 발생(generic array creation)합니다.

```java
import java.util.*;

public class Main {
    public static void main(String args[]) {
        List<String>[] stringLists = new List<String>[1];
        Object[] objects = stringLists;
        objects[0] = new ArrayList<Integer>();
    }
}
```

```bash
/Main.java:5: error: generic array creation
        List<String>[] stringLists = new List<String>[1];
                                     ^
1 error
```

그런데 만약 error가 발생하지 않는다면? `new ArrayList<Integer>();` 로 생성한 다른 타입의 인스턴스가 대입 가능해집니다.

Generic restriction 문서([링크](https://docs.oracle.com/javase/tutorial/java/generics/restrictions.html#createArrays))에서 배열 생성이 불가능한 내용을 확인할 수 있습니다.

살펴본 것과 같이 Generic Type 인스턴스와 배열은 Type Erasure로 인해 생성할 수 없는 제한이 걸려있습니다.

나머지 Type Erasure에 대한 내용은 크게 연관은 없지만 본 김에 한 번 살펴보겠습니다.

#### 형 변환 및 bridge method

문서([Effects of Type Erasure and Bridge Methods](https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html))내용을 간단하게 살펴보겠습니다.

아래와 같은 Generic 클래스가 있다고 가정합니다.

```java
public class Node<T> {

    public T data;

    public Node(T data) { this.data = data; }

    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

그리고 아래와 같은 코드를 실행한다고 가정합니다.

```java
MyNode mn = new MyNode(5);
Node n = mn;
n.setData("Hello");
Integer x = mn.data;
```

컴파일러에 의해 Type Erasure 를 거친 후에는 아래와 같이 코드가 변경됩니다.

```java
MyNode mn = new MyNode(5);
Node n = mn;            // A raw type - compiler throws an unchecked warning
                        // Note: This statement could instead be the following:
                        //     Node n = (Node)mn;
                        // However, the compiler doesn't generate a cast because
                        // it isn't required.
n.setData("Hello");     // Causes a ClassCastException to be thrown.
Integer x = (Integer)mn.data; 
```

참고로 [raw type](https://docs.oracle.com/javase/tutorial/java/generics/rawTypes.html)은 Generic 클래스 또는 인터페이스인데 타입을 지정하지 않은 것을 의미합니다.

generic type 인 mn.data의 경우 자동으로 `형 변환`이 발생하는 것을 확인할 수 있습니다. 그런데 중요한 것은 `ClassCastException`이 던져지는 것입니다.

일반적인 코드라면 아래와 같이 incompatible types 라는 error가 발생합니다.

```bash
jshell> public void setData(Integer data) {
   ...>         System.out.println("MyNode.setData");
   ...> }
|  created method setData(Integer)

jshell> setData("limvik");
|  Error:
|  incompatible types: java.lang.String cannot be converted to java.lang.Integer
|  setData("limvik");
|          ^------^
```

Error가 아닌 ClassCastException이 던져지는 이유는 bridge method 때문입니다.

앞서 본 Node 클래스와 MyNode 클래스가 컴파일러에 의해 Type Erasure 프로세스를 거치고나면, 아래와 같이 변환됩니다.

```java
public class Node {

    public Object data;

    public Node(Object data) { this.data = data; }

    public void setData(Object data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

public class MyNode extends Node {

    public MyNode(Integer data) { super(data); }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

MyNode 클래스의 setData 메서드의 인수는 Integer 타입인데, Node 클래스의 setData는 Object 타입이므로 오버라이드 메서드가 아니게 됩니다.

그래서 Java compiler는 이 문제를 해결하기 위해 bridge method를 만들어 오버라이드하면서 형 변환을 수행합니다.

```java
class MyNode extends Node {

    // Bridge method generated by the compiler
    //
    public void setData(Object data) {
        setData((Integer) data);
    }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }

    // ...
}
```

bridge method를 만들어 오버라이드하고, 실제로는 기존의 메서드를 호출하면서 `setData((Integer) data);` 와 같이 형 변환을 수행하여, `setData("limvik");` 와 같이 다른 타입을 입력하면 ClassCastException이 던져지게됩니다.

## Outro

지금까지 살펴본 자료를 통해 제목에 대한 답을 정리해보자면 Generic Type 의 인스턴스 생성이나 배열 생성을 막아둔 것은 이를 허용할 경우 `Type Erasure`로 인해서 Generics의 목적 중 하나인 컴파일 타임의 타입 체크를 완전히 버리는 것과 같기 때문입니다. (하지만 제가 Generics를 모두 훑어본 것은 아니라 추가적인 이유가 있지도 않을까 생각이 됩니다.)

그런데 이게 사용하는 입장에서는 왜 굳이 정보를 지워버려서 불편하게 하는지 이해하기 어려워서, 다수의 Java 개발자들이 불만을 갖고 있는 것 같습니다.

그래서 Java Language Architect 인 Brian Goetz가 2020년에 이에 대한 반박 글([Background: How We Got the Generics We Have(Or, how I learned to stop worrying and love erasure)](https://openjdk.org/projects/valhalla/design-notes/in-defense-of-erasure))을 올립니다.

글에는 Java 개발자들이 Type Erasure에 대해 오해하고있는 점이 많고, Generics를 구현하는 방식에도 다양한 방법이 있는데 Java에서는 왜 이러한 방식을 택했는지 등 다양한 내용이 나오지만, 이해하지 못하는 용어들이 많아서 간단하게 살펴보기만 했습니다. 프로그래밍 언어 아키텍트까지는 아직 생각이 없어서, 더 깊게 파보는 것은 취업 이후에 하는게 좋겠습니다. 지금은 Generics 포함해서 Java의 다른 기능들을 잘 사용하는 방법을 더 깊게 파보는 것을 우선순위에 두는게 맞다고 생각되어 여기서 마무리합니다.
