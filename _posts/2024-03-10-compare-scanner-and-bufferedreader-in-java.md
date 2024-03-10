---
layout: post
title: Java, Scanner와 BufferedReader 비교
categories:
- Java
tags:
- Java
- Scanner
date: 2024-03-10 22:41 +0900
---
## Intro

프로그래머스 문제를 풀때는 입력을 직접 처리하지 않다보니 [`Scanner`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/Scanner.java)나 [`BufferedReader`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/io/BufferedReader.java)를 사용할 일이 없습니다. 그러다가 백준으로 넘어오면 입력을 처리하기 위해 편리한 Scanner를 사용하다가, Scanner를 사용하면 시간초과가 발생하는 문제(예. [15552번-빠른 A+B](https://www.acmicpc.net/problem/15552))를 만나 BufferedReader로 풀고는 합니다.

어떤 차이가 있길래 성능 차이가 발생하는지 궁금해서 코드를 살펴보았습니다. 코드는 `openjdk 17`을 기준으로 합니다.

## 생성자 살펴보기

제가 주로 사용하는 생성자(Constructor)에서 어떻게 초기화하고 있는지 보겠습니다.

### Scanner

Scanner의 생성자 코드([GitHub 링크](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/Scanner.java#L533))와 관련 코드를 보겠습니다.

```java
private static final int BUFFER_SIZE = 1024;

private static Pattern WHITESPACE_PATTERN = Pattern.compile(
                                                "\\p{javaWhitespace}+");

private Scanner(Readable source, Pattern pattern) {
    assert source != null : "source should not be null";
    assert pattern != null : "pattern should not be null";
    this.source = source;
    delimPattern = pattern;
    buf = CharBuffer.allocate(BUFFER_SIZE);
    buf.limit(0);
    matcher = delimPattern.matcher(buf);
    matcher.useTransparentBounds(true);
    matcher.useAnchoringBounds(false);
    useLocale(Locale.getDefault(Locale.Category.FORMAT));
}

public Scanner(InputStream source) {
    this(new InputStreamReader(source), WHITESPACE_PATTERN);
}
```

`util` 패키지에 있는 Scanner 답게 편의를 위해, 직접 설정하지 않아도 기본값으로 구분자(delimiter)를 공백으로 지정합니다. 그리고 Locale까지 지정합니다.

키보드 입력을 받기 위해 Scanner를 사용할 때 저는 아래와 같이 초기화 하는데, System 클래스의 [`in`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/lang/System.java#L129) 변수는 `InputStream` 타입임을 알 수 있습니다.

```java
Scanner sc = new Scanner(System.in);

// System.java
public static final InputStream in = null;
```

BufferedReader의 인스턴스를 생성할 때는 InputStreamReader를 인수로 넘겨주면서 InputStreamReader의 인수로는 System클래스의 in 변수를 넘겨주는데, 한 꺼풀만 까보면 초기화 하는 모습이 비슷해 보입니다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
```

### BufferedReader

BufferedReader의 생성자 코드([GitHub 링크](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/io/BufferedReader.java#L116))와 관련 코드를 살펴보겠습니다.

```java
private Reader in;

private char[] cb;
private int nChars, nextChar;

private static int defaultCharBufferSize = 8192;

public BufferedReader(Reader in, int sz) {
    super(in);
    if (sz <= 0)
        throw new IllegalArgumentException("Buffer size <= 0");
    this.in = in;
    cb = new char[sz];
    nextChar = nChars = 0;
}

public BufferedReader(Reader in) {
    this(in, defaultCharBufferSize);
}
```

Scanner가 구분자 설정 등 편의를 위한 설정코드가 많이 있는 반면, 입출력을 처리하기 위한 `io` 패키지에 있는 BufferedReader 답게 부가적인 일들은 하지 않고 있습니다.

눈에 띄는 점이라면 Scanner 대비 `Buffer Size의 기본값이 8배(1024 * 8 = 8192)`나 되며, Scanner는 Buffer Size를 변경할 수 없는 반면에 BufferedReader는 `Buffer Size를 변경 가능`합니다.

Intro에서 언급했던 문제는 테스트 케이스가 최대 1,000,000개 이고, 두 수의 최대 값이 1000 입니다. `1000 1000` 과 같은 문자열은 jdk 17에서 각 문자가 1byte 가 되어 최악의 경우 총 9byte x 1,000,000 = `9MB` 정도의 데이터를 처리해야 합니다.

BufferedReader의 Buffer Size가 더 커서 한 번에 처리할 수 있는 데이터가 많으니 Scanner 보다는 확실히 빠르게 처리할 수 있을 것으로 보입니다.

하지만 BufferedReader는 Thread Safe하므로 성능 면에서 악영향을 줄 수 있는 부분도 있습니다. 

상위 클래스로 입력 스트림을 보내기 위해 `super(in);`과 같은 코드가 있는 것을 볼 수 있습니다. BufferedReader는 추상 클래스 [`Reader`](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/io/Reader.java)를 상속받고 있습니다.

Reader의 생성자 코드([GitHub 링크](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/io/Reader.java#L166))를 보면, 파라미터 이름도 lock으로되어 있고, 주석을 살펴보면 Thread Safe 하게 처리해야 한다는 사실을 알 수 있습니다.

```java
/**
 * The object used to synchronize operations on this stream.  For
 * efficiency, a character-stream object may use an object other than
 * itself to protect critical sections.  A subclass should therefore use
 * the object in this field rather than {@code this} or a synchronized
 * method.
 */
protected Object lock;

/**
 * Creates a new character-stream reader whose critical sections will
 * synchronize on the reader itself.
 */
protected Reader() {
    this.lock = this;
}

/**
 * Creates a new character-stream reader whose critical sections will
 * synchronize on the given object.
 *
 * @param lock  The Object to synchronize on.
 */
protected Reader(Object lock) {
    if (lock == null) {
        throw new NullPointerException();
    }
    this.lock = lock;
}
```

Thread Safe하지 않은 Scanner 대비 약간의 성능 저하가 있을 수 있지만, 입력의 크기가 클수록 Buffer Size에 의한 성능 차이에 비하면 미미할 것으로 예상됩니다. 이 성능 차이를 측정해볼 수 있는 도구가 뭐가 있을지 찾아봐야겠습니다. Profiler 를 얼핏 들은거 같기도 한데, 맞으려나 모르겠습니다.

생성자에서는 Thread Safe 하다는 것과 Buffer Size의 차이를 보았습니다. 이어서 한 줄을 읽는 BufferedReader의 readLine() 메서드와 Scanner의 nextLine() 메서드를 살펴보겠습니다.

## nextLine() vs readLine()

먼저 Scanner의 nextLine() 코드와 관련 코드([GitHub 링크](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/util/Scanner.java#L1643))를 살펴보겠습니다.

### Scanner의 nextLine()

Scanner는 개행 문자를 패턴으로 지정하여 정규표현식으로 한 줄의 끝을 찾은 후, 문자열을 잘라내는 방식임을 볼 수 있습니다.

```java
private static volatile Pattern linePattern;
private static final String LINE_SEPARATOR_PATTERN =
                                       "\r\n|[\n\r\u2028\u2029\u0085]";
private static final String LINE_PATTERN = ".*("+LINE_SEPARATOR_PATTERN+")|.+$";

private static Pattern linePattern() {
    Pattern lp = linePattern;
    if (lp == null)
        linePattern = lp = Pattern.compile(LINE_PATTERN);
    return lp;
}

public String nextLine() {
    modCount++;
    if (hasNextPattern == linePattern())
        return getCachedResult();
    clearCaches();

    String result = findWithinHorizon(linePattern, 0);
    if (result == null)
        throw new NoSuchElementException("No line found");
    MatchResult mr = this.match();
    String lineSep = mr.group(1);
    if (lineSep != null)
        result = result.substring(0, result.length() - lineSep.length());
    if (result == null)
        throw new NoSuchElementException();
    else
        return result;
}
```

### BufferedReader의 readLine()

BufferedReader의 readLine() 코드와 관련 코드([GitHub 링크](https://github.com/openjdk/jdk17u/blob/master/src/java.base/share/classes/java/io/BufferedReader.java#L316))를 보겠습니다.

`synchronized` 키워드를 이용하여 critical section을 설정한 것을 볼 수 있습니다. 그리고 Label을 이용해서 loop를 탈출하는 것을 볼 수 있는데, 기본서에서 되도록 쓰지 말라는 문구를 본 이후에 실제로 사용한 것은 처음 봅니다.

그리고 Scanner와는 달리 Character를 하나씩 읽으면서 `StringBuilder`를 이용해 문자열을 만들고, 개행 문자를 만나면 종료하는 것을 볼 수 있습니다.

```java
String readLine(boolean ignoreLF, boolean[] term) throws IOException {
    StringBuilder s = null;
    int startChar;

    synchronized (lock) {
        ensureOpen();
        boolean omitLF = ignoreLF || skipLF;
        if (term != null) term[0] = false;

    bufferLoop:
        for (;;) {

            if (nextChar >= nChars)
                fill();
            if (nextChar >= nChars) { /* EOF */
                if (s != null && s.length() > 0)
                    return s.toString();
                else
                    return null;
            }
            boolean eol = false;
            char c = 0;
            int i;

            /* Skip a leftover '\n', if necessary */
            if (omitLF && (cb[nextChar] == '\n'))
                nextChar++;
            skipLF = false;
            omitLF = false;

        charLoop:
            for (i = nextChar; i < nChars; i++) {
                c = cb[i];
                if ((c == '\n') || (c == '\r')) {
                    if (term != null) term[0] = true;
                    eol = true;
                    break charLoop;
                }
            }

            startChar = nextChar;
            nextChar = i;

            if (eol) {
                String str;
                if (s == null) {
                    str = new String(cb, startChar, i - startChar);
                } else {
                    s.append(cb, startChar, i - startChar);
                    str = s.toString();
                }
                nextChar++;
                if (c == '\r') {
                    skipLF = true;
                }
                return str;
            }

            if (s == null)
                s = new StringBuilder(defaultExpectedLineLength);
            s.append(cb, startChar, i - startChar);
        }
    }
}

public String readLine() throws IOException {
    return readLine(false, null);
}
```

Scanner와 달리 정규표현식을 사용하지 않으며, 문자열을 불러와서 잘라내는 방식이 아니기 때문에 느낌적인 느낌으로는 성능이 좋을 수 밖에 없겠구나라는 생각이 듭니다.

객관적인 평가를 위해 성능 측정을 하고 싶은데, 이를 세세하게 측정하기 위한 도구를 학습하게 되면 업데이트 해보겠습니다. 지금은 그다지 우선순위가 높은 작업은 아니라 언제가 될지는 모르겠습니다.

## 보너스. Close를 해야할까?

예전에 Scanner나 BufferedReader 사용하고 Close 하지 않으면 감점당할 수 있다고 들었는데, 실제로는 굳이 Close할 필요가 없습니다.

과거에 Java 처음 배우고서 연습삼아 Console 프로그램 만들면서 try-with-resources 구문으로 Auto Close 했다가 입력이 안되는데, 안되는 이유를 못찾아 고생한 적이 있습니다.

그래서 [`Java에서 new Scanner(System.in);로 생성한 인스턴스는 언제 close() 메서드를 호출해야할까?`](/posts/when-should-i-close-the-scanner-class/) 라는 글도 작성했습니다. 글을 작성하면서 참고한 공식 문서에서는 굳이 기본값으로 열어둔 키보드 스트림 `System.in` 를 포함해서 대부분의 스트림 인스턴스는 닫을 필요가 없다고 언급하고 있는 것을 확인했습니다.

그런데 아직 어떤 차이점을 바탕으로 반드시 Close 해야 하는 것과 Close 하지 않아도 되는 것을 구분할 수 있는지 명확하게 파악하지 못했습니다. 확실히 답할 수 있기 전까지는 일단 코딩테스트에서는 Flush 하고 Close 하는게 좋아보입니다.

## Outro

Scanner와 BufferedReader를 코드 측면에서 살펴보긴 했지만, 실제 각 코드로 인한 성능 차이를 비교해볼 방법은 아직 잘 몰라서 아쉬운 글이 됐습니다.

아쉽긴 하지만, 코드 측면에서 궁금했던 분들께 도움이 되었으면 좋겠습니다.
