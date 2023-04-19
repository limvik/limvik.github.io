---
layout: post
title: Java Scanner 클래스에서 정규표현식 사용하기
categories:
- Java
tags:
- Java
- Scanner
- Pattern
- Regex
date: 2023-04-19 21:41 +0900
---
## Intro

[이전 글](https://limvik.github.io/posts/database-design-1-4-physical-data-modeling/)에서 물리 데이터 모델링을 하고, 조금씩이나마 구현하고 있습니다. (설계한게 무슨 소용이었나 싶을 정도로 뜯어 고쳐야 할 것이 많이 보입니다.)

Java를 이용해서 CLI(Command Line Interface)로 만들다보니, 사용자 입력을 받기 위해 간편하게 Scanner 클래스(Class)를 사용하였습니다.

그런데 예외처리를 하려고보니 제가 알고 있는 try-catch-finally 말고는 방법이 없나 의문이 들었습니다. 그러다 Scanner 클래스에서 `next()` 또는 `hasNext()` 메서드(method)의 인수로 정규표현식을 지정한 `Pattern` 클래스를 사용할 수 있다는 것을 알게됐습니다. try-catch-finally 보다 코드가 간결하고 신경써야 할 예외가 줄어 개인적으로는 굉장히 추천합니다.

## Code

아래는 제 코드 일부를 가져왔습니다.

```java
import java.util.Scanner;
import java.util.regex.Pattern;

//...

Scanner stdIn = new Scanner(System.in);

// 1부터 메뉴 마지막 번호까지 정수에 대한 정규표현식 지정
Pattern pattern = Pattern.compile("[1-" + view.getMenuLength() + "]");

// 입력 받기 및 입력 확인
while (!stdIn.hasNext(pattern)) {
    // 오류 문장 출력
    view.printError();
    // 잘못된 입력 버퍼에서 제거
    stdIn.nextLine();
}
    
// 선택된 메뉴번호 저장
int menu = stdIn.nextInt();

```

제가 알고있는 try-catch-finally 구문을 이용한 예외처리를 하려면, 정수가 아닌 경우부터 시작해서 메뉴의 범위를 벗어났을 경우 등 여러가지를 처리해줘야 했습니다. 그런데 `Pattern` 클래스를 이용하면 아주 간단하게 정규표현식과 일치하지 않는 경우에는 다시 입력을 받음으로써 해결할 수 있었습니다.

초보자라도 바로 이해가 될 가능성이 높지만, Pattern 클래스를 사용한 부분만 간단히 보겠습니다. 

아래를 보시면 Pattern 클래스를 선언하고, compile 메서드의 인수에 정규표현식을 지정하였습니다. 제가 만들고 있는 어플리케이션의 현재 메뉴가 2개이므로, `[1-2]` 와 같이 정수 1과 2만 허용하는 정규표현식이 됩니다. 메뉴가 늘어날 가능성도 고려하여, 메뉴 갯수 증가에 따른 영향이 없도록 하였습니다.

```java
Pattern pattern = Pattern.compile("[1-" + view.getMenuLength() + "]");
```

하지만 메뉴 길이가 10 이상이 될 것에 대한 고려는 없는 코드라는 사실이 뒤늦게 떠올랐습니다.

다음으로 while 문의 조건으로 사용된 pattern 입니다. pattern과 일치하지 않을 경우 `false`가 반환되기 때문에 pattern과 일치하는 값을 입력할 때까지 반복해서 입력을 받을 수 있습니다.

```java
while (!stdIn.hasNext(pattern)) {
```
Pattern 클래스 공식 문서([링크](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/regex/Pattern.html))를 보시면 다양한 정규표현식 예제를 보실 수 있습니다.
> Java에서 `[0-9]` 보다는 `\d` 를 사용한다는 점은 처음알게 됐습니다.

## Outro

글은 마무리 하지만 chat GPT 한테 다른 질문하다 얻어걸려서 알게 된 내용이라, 어쩌다 얻어걸린 과정도 참고 삼아 아래에 정리했습니다.

## 잡설: Pattern 클래스 발견 과정

사실 try-catch-finally 를 대체할 방법을 찾고는 있었지만, Scanner 클래스 관련해서 다른 문제에 대한 해결방법을  찾다가 Pattern 클래스 사용하는 방법을 발견했습니다.

Scanner 클래스가 한 번 close() 하고나면, 다시 Scanner 클래스 인스턴스를 생성해도 입력을 받아오지 못하는 것 때문에 이전에 많은 시간을 보냈던 적이 있어서 Scanner 클래스를 잘 사용하는 `패턴`이 있는지 chat GPT에 물어보다가 알게 됐습니다. 아무래도 `패턴` 이라는 단어가 트리거가 됐던 것 같습니다.

아래는 Scanner 클래스 사용 패턴을 물었더니 알려줬던 코드입니다. 코드는 굳이 안넣을까 하다가 앞에서 제시한 코드가 더러운 부분이 있어서 이해 못하신 분 있을까봐 붙여넣습니다.

```java
import java.util.Scanner;
import java.util.regex.Pattern;

public class ScannerExample {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("이름을 입력하세요: ");
        String name = scanner.nextLine();

        System.out.print("나이를 입력하세요: ");
        int age = scanner.nextInt();

        // 패턴 사용 예시: 이메일 주소 입력 받기
        Pattern emailPattern = Pattern.compile("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,6}$");
        System.out.print("이메일을 입력하세요: ");
        while (!scanner.hasNext(emailPattern)) {
            System.out.println("올바른 이메일 형식이 아닙니다. 다시 입력해주세요.");
            scanner.nextLine();
        }
        String email = scanner.next(emailPattern);

        scanner.close();

        System.out.println("이름: " + name);
        System.out.println("나이: " + age);
        System.out.println("이메일: " + email);
    }
}
```

이렇게 소 뒷걸음 치다 쥐잡은 느낌으로 찾아보려던걸 찾았습니다.

### 해결하지 못한 다른 문제

하지만 원래 질문 목적인 Scanner 클래스를 어떻게 잘 쓸 방법은 없는지는 제대로 답변을 안주고, 오류나는 코드를 줍니다.

```java
package com.limvik;

import java.util.Scanner;

public class ScannerCloseExample {

    public static void main(String[] args) {
        Scanner scanner1 = new Scanner(System.in);

        System.out.print("첫 번째 숫자를 입력하세요: ");
        int num1 = scanner1.nextInt();
        scanner1.close();

        // 다시 입력을 받기 위해 새로운 Scanner 객체를 생성
        Scanner scanner2 = new Scanner(System.in);

        System.out.print("두 번째 숫자를 입력하세요: ");
        int num2 = scanner2.nextInt();
        scanner2.close();

        int sum = num1 + num2;
        System.out.println("두 숫자의 합: " + sum);
    }
}
```

![Scanner Class Error](/assets/img/2023-04-19-scanner-class-with-regex/2023-04-19-scanner-close-error.png)

그래서 일단은 싱글턴(Singleton) 패턴을 이용해서 어플리케이션 실행하는 동안 Scanner 클래스 인스턴스를 한 개만 유지하도록 했습니다.

```java
import java.util.Scanner;

public class InputController {
    private static InputController instance;
    private Scanner scanner;

    private InputController() {
        scanner = new Scanner(System.in);
    }

    public static InputController getInstance() {
        if (instance == null) {
            instance = new InputController();
        }
        return instance;
    }

    public Scanner getScanner() {
        return scanner;
    }

    public void closeScanner() {
        scanner.close();
    }
}
```

Scanner 클래스 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Scanner.html))나 Scanning 문서([링크](https://docs.oracle.com/javase/tutorial/essential/io/scanning.html))를 봐도 이에 대한 언급이 없습니다. 굉장히 기본적인 내용인데 제가 뭔가 놓치고 있는건가 고민이됩니다. Java의 정석에서는 close()를 안하는데, 애초에 close()를 안해도 별 상관이 없는데 쓸데없이 시간 쓰는건가 싶기도 하네요. 일단은 여기까지 조사하고 계속 구현해 봐야겠습니다.
