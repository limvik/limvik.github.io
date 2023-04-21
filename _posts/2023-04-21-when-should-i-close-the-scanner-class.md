---
layout: post
title: Java에서 new Scanner(System.in);로 생성한 인스턴스는 언제 close() 메서드를 호출해야할까?
categories:
- Java
tags:
- Java
- Scanner
- Stream
date: 2023-04-21 20:59 +0900
---
## Intro
제목에 대한 답을 먼저 말씀드리자면, System 클래스의 in을 변경한 적이 없을 경우 키보드 입력 받을 일이 없거나 프로그램 종료할 때 입니다.

[이전 글](https://limvik.github.io/posts/java-scanner-class-with-regex/)에서 잡설을 하면서 close() 에 대해 이해가 안돼서 툴툴 거렸는데, 궁금해서 Bing chat, chat GPT 와의 협업(?)으로 찾아봤습니다. Bard는 헛소리 해서 아쉽게도 협업에 참여하지 못했습니다...

## close() 메서드 호출하면 사용 불가능

앞서 언급한 이전 글에서도 적었지만, Scanner 클래스의 close() 메서드를 호출하고 나면, `System.in`을 인수로 갖는 Scanner 클래스의 새로운 인스턴스를 만들어도 입력을 받을 수 없습니다.

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
![오류 발생 화면](/assets/img/2023-04-19-scanner-class-with-regex/2023-04-19-scanner-close-error.png)

검색을 해보니 Scanner 클래스의 close() 메서드를 호출하면, System.in 까지 닫아서 그렇다고 합니다. 하지만 저는 그게 무슨 소린지 이해가 안돼서 더 찾아봤습니다.

## Scanner를 close 했는데 왜 System.in 까지 닫아버리는걸까?

Scanner 공식 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Scanner.html))를 보면, Closeable 인터페이스(Interface)를 implemented 하고 있고, Closeable 공식 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Closeable.html))를 보면 아래와 같이 나와 있습니다.

> Closes this stream and releases any system resources associated with it.

`Stream을 닫아`버리고 관련된 모든 시스템 자원을 해제하는 메서드라고 합니다.

인터페이스에서 그렇게 정의를 했으니 클래스에서 구현하면서 Stream을 닫아버리도록 구현을 해야겠죠. 아래는 Scanner 클래스 코드([Github 링크](https://github.com/openjdk/jdk/blob/0deb648985b018653ccdaf193dc13b3cf21c088a/src/java.base/share/classes/java/util/Scanner.java)) 일부 입니다.

Readable 인터페이스([문서 링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/class-use/Readable.html))를 구현한 source라면 close() 메서드를 호출하고 있는 것을 볼 수 있습니다.

```java
public final class Scanner implements Iterator<String>, Closeable {
    // ...
	
    // The input source 341번 줄
    private Readable source;
    
    // ... 1176번 줄
    public void close() {
        if (closed)
            return;
        if (source instanceof Closeable) {
            try {
                ((Closeable)source).close();
            } catch (IOException ioe) {
                lastException = ioe;
            }
        }
        sourceClosed = true;
        source = null;
        closed = true;
     }
     // ...
}
```

문서를 참고해보면 java.io에서 Readable 인터페이스를 구현한 클래스로는 BufferedReader, CharArrayReader, FileReader, FilterReader, InputStreamReader, LineNumberReader, PipedReader, PushbackReader, Reader, StringReader 가 있습니다.

그리고 System 클래스 공식 문서([링크](https://docs.oracle.com/javase/8/docs/api/java/lang/System.html#in))의 in 에 대한 설명을 보면 아래와 같습니다.

> public static final `InputStream` in
> 
> The "standard" input stream. This stream is already open and ready to supply input data. Typically this stream corresponds to keyboard input or another input source specified by the host environment or user.

System.in 은 `InputStream` 임을 알 수 있습니다. 이미 열려있고 기본적으로 키보드 입력에 연결되어 있다고 설명하고 있습니다.

이러한 InputStream을 받는 Scanner 클래스의 생성자(Constructor)는 아래와 같습니다.

```java
// 570번 줄
public Scanner(InputStream source) {
    this(new InputStreamReader(source), WHITESPACE_PATTERN);
}
```

new Scanner(System.in); 과 같이 호출하면 다시 InputStreamReader의 인수로 전달합니다.

그럼 또 다시 InputStreamReader의 공식 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/InputStreamReader.html#close%28%29))의 close() 메서드에 대한 설명을 보면 아래와 같습니다.

> public void close()
>           throws IOException
>
> Description copied from class: Reader
> 
> Closes the stream and releases any system resources associated with it. Once the stream has been closed, further read(), ready(), mark(), reset(), or skip() invocations will throw an IOException. Closing a previously closed stream has no effect.

Stream이 close 된 이후에 추가적인 작업을 하려고 하면 IOException이 throw 된다고 설명합니다.

정리하자면, Scanner 클래스에서 close() 메서드를 호출하면 System.in 에서 참조하고 있는 InputStream을 닫아버리게 되는 것입니다.

그럼 이제 System.in 까지 닫은 이유와 System.in이 참조하던 Stream을 닫아버려서 재사용할 수 없다 것은 이해가 되는데...  `닫은걸 다시 열어주면 되는거 아닌가?` 하는 의문이 생깁니다.

## Stream을 다시 열면 되는거 아닌가?

System.in 을 초기화하는 코드([Github 링크](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/lang/System.java))를 확인해 보면 아래와 같습니다.

```java
public final class System {
    // ...137번 줄
    public static final InputStream in = null;
    
    // ...232번 줄
    public static void setIn(InputStream in) {
        checkIO();
        setIn0(in);
    }
    
    // ...344번 줄
    private static native void setIn0(InputStream in);
    
    // ...2141번 줄
    /**
     * Initialize the system class.  Called after thread initialization.
     */
    private static void initPhase1() {
        // ...2181번 줄
        FileInputStream fdIn = new FileInputStream(FileDescriptor.in);
        FileOutputStream fdOut = new FileOutputStream(FileDescriptor.out);
        FileOutputStream fdErr = new FileOutputStream(FileDescriptor.err);
        initialIn = new BufferedInputStream(fdIn);
        setIn0(initialIn);
        // ...
    }
}
```

System.in 의 초기값은 thread가 초기화된 후에 FileDescriptor 클래스에서 받아오는 것을 알 수 있습니다. 

System.in에 별도로 InputStream을 설정하지 않았다면, Scanner 클래스 생성자 인수로 System.in 을 사용하여 인스턴스를 만들었을 때 close() 메서드 호출 시 실제로는 FileDescriptor.in에 설정된 Stream을 닫는 것이라는 걸 알 수 있습니다.

아래처럼 Scanner를 try-with-resources 구문으로 close 하고, FileDescriptor.in을 이용해 입력받는 코드를 실행하면 `java.io.IOException: Stream Closed` 라는 메시지를 볼 수 있습니다.

```java
package com.limvik;

import java.io.BufferedReader;
import java.io.FileDescriptor;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;

public class ScannerCloseExample {

    public static void main(String[] args) {
        try(Scanner sc = new Scanner(System.in)) {
            System.out.print("첫 번째 숫자를 입력하세요: ");
            sc.nextInt();
        }

        System.out.print("이름을 입력하세요: ");

        try (FileInputStream customIn = new FileInputStream(FileDescriptor.in);
             BufferedReader reader = new BufferedReader(new InputStreamReader(customIn))) {
            String name = reader.readLine();
            System.out.println("안녕하세요, " + name + "님!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
![Error Message](/assets/img/2023-04-21-when-should-i-close-the-Scanner-class/2023-04-21-file-description-stream-closed.png)

FileDescriptor 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/io/FileDescriptor.html#in))의 FileDescriptor.in 설명에도 보통 System.in 을 쓴다는 언급만 있을 뿐 별 내용은 없습니다.

> A handle to the standard input stream. Usually, this file descriptor is not used directly, but rather via the input stream known as System.in.

그럼 또... FileDescriptor의 in은 어떻게 초기화를 하고 있나 코드([Github 링크](https://github.com/openjdk/jdk/blob/0deb648985b018653ccdaf193dc13b3cf21c088a/src/java.base/share/classes/java/io/FileDescriptor.java))를 보면 아래와 같습니다.

```java
public final class FileDescriptor {
    // ... 131번 줄
    /**
     * Used for standard input, output, and error only.
     * For Windows the corresponding handle is initialized.
     * For Unix the append mode is cached.
     * @param fd the raw fd number (0, 1, 2)
     */
    private FileDescriptor(int fd) {
        this.fd = fd;
        this.handle = getHandle(fd);
        this.append = getAppend(fd);
    }

    /**
     * A handle to the standard input stream. Usually, this file
     * descriptor is not used directly, but rather via the input stream
     * known as {@code System.in}.
     *
     * @see     java.lang.System#in
     */
    public static final FileDescriptor in = new FileDescriptor(0);

    // ... 231번 줄
    /*
     * On Windows return the handle for the standard streams.
     */
    private static native long getHandle(int d);

    /**
     * Returns true, if the file was opened for appending.
     */
    private static native boolean getAppend(int fd);
}
```

private 메서드라 직접 호출할 수는 없겠습니다. 직접 native 코드를 만들지 않는 이상, 다시 Stream을 여는 것은 힘들어 보이네요. 그냥 close() 안하면 되는걸 굳이 그런 노력까지 할 필요는 없겠죠.

## close() 메서드 호출 해야하는 경우

Stream 인터페이스 문서([링크](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html))를 확인해보면 아래와 같은 문장이 있습니다.

> Streams have a BaseStream.close() method and implement AutoCloseable. Operating on a stream after it has been closed will throw `IllegalStateException`. **Most stream instances do not actually need to be closed after use**, as they are backed by collections, arrays, or generating functions, which require no special resource management. **Generally, only streams whose source is an IO channel, such as those returned by Files.lines(Path), will require closing.** If a stream does require closing, it `must` be opened as a resource within a `try-with-resources statement or similar control structure` to ensure that it is closed promptly after its operations have completed.

대부분의 Stream 인스턴스는 사용 후에 close 할 필요가 없고, 보통 [Files](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/nio/file/Files.html).lines(Path)에 의해 반환되는 것과 같이 source가 IO channel 일 때만 닫아주면 된다고 합니다.

## Outro

close 안해도 된다는 Stream 문서를 먼저 봐서, 그냥 그렇구나 하고 넘어가려다가 갑자기 호기심이 생겨서 여기저기 들쑤시고 다녔습니다. 검색하다보니 같은 궁금증을 가지셨던 분들이 꽤 있었던거 같은데 조금이나마 도움이 됐으면 좋겠습니다.

Scanner 클래스를 close 안하면 IDE(Integrated Development Environment)에서 자꾸 warning을 띄워서 신경쓰이게 만드는데, Console의 여러 곳에서 키보드 입력을 받아야 한다면 싱글턴(Singleton) 패턴으로 Scanner 클래스 인스턴스를 1개만 유지하는게 제가 아는 선에서는 가장 나은 방법으로 보입니다. 이제 Javascript 열심히 배우고 있어서 Java로 키보드 입력 받을 일이 얼마나 있을지는 모르겠지만 더 좋은 방법을 알게 되면 추가로 글 작성해 보겠습니다.

## 참고자료 정리

- [Scanner close 반드시 해야할까?](https://ikjo.tistory.com/131)
- [Scanner 에 Console 사용해서 close 후에 다시 Scanner 생성](https://m.blog.naver.com/shwsun/40172530715)
- [Stream](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Stream.html)
- [System](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html)
- [System.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/lang/System.java)
- [Scanner](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Scanner.html)
- [Scanner.java](https://github.com/openjdk/jdk/blob/0deb648985b018653ccdaf193dc13b3cf21c088a/src/java.base/share/classes/java/util/Scanner.java)
- [Closeable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Closeable.html)
- [Readable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/class-use/Readable.html)
- [FileDescriptor](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/io/FileDescriptor.html)
- [FileDescriptor.java](https://github.com/openjdk/jdk/blob/0deb648985b018653ccdaf193dc13b3cf21c088a/src/java.base/share/classes/java/io/FileDescriptor.java)
- [Files](https://docs.oracle.com/en/java/javase/17/docs/api//java.base/java/nio/file/Files.html)
- [Java Language Specification (JLS), 12.4.2. Detailed Initialization Procedure](https://docs.oracle.com/javase/specs/jls/se17/html/jls-12.html#jls-12.4.2)
- [Java Virtual Machine Specification (JVMS), 5.5. Initialization](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-5.html#jvms-5.5)

## 추가되는 질문들

- File IO 후에 close() 하지 않으면 무슨 문제가 생기는걸까?
- File Description이 정확히 뭘까?
