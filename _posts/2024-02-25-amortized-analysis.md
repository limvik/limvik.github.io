---
layout: post
title: 분할 상환 분석(Amortized Analysis)
categories:
- Algorithms
tags:
- Algorithms
date: 2024-02-25 18:30 +0900
---
## Intro

Java의 [ArrayList](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayList.java)는 데이터 추가 시 공간이 부족한 경우 크기를 기존 배열 크기의 절반만큼 증가시킵니다(증가 시 배열 최대 크기를 초과하는 경우 제외).

> Java 6까지는 [ArrayList](https://github.com/openjdk/jdk6/blob/master/jdk/src/share/classes/java/util/ArrayList.java#L180)에서 `int newCapacity = (oldCapacity * 3)/2 + 1;` 과 같은 수식을 사용했는데, 3을 곱할 때 Overflow가 발생할 수도 있고, bit 연산에 비해 연산도 많습니다.

데이터를 추가하는 [add](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayList.java#L482) 메서드 중 하나를 보면, 배열이 가득 찬 경우 [grow](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayList.java#L232) 메서드를 호출합니다.

```java

private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}

private Object[] grow() {
    return grow(size + 1);
}
```

> 코드 참고  
> ArraysSupport.[newLength](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/jdk/internal/util/ArraysSupport.java#L735)()  
> Arrays.[copyOf](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/Arrays.java#L3480)()

데이터 추가 시 공간이 부족한 경우 `Arrays.copyOf`를 통해 새로운 배열을 만들어 기존 배열을 복사해야 하므로, 흔히 사용하는 최악의 경우를 생각해보면 데이터 추가 시 `O(n)`의 시간복잡도를 갖습니다.

배열의 최대 크기인 2의 32제곱(=4,294,967,296)개 만큼의 데이터를 하나씩 추가한다고 할 때, ArrayList는 기본값인 경우 배열의 크기를 10부터 시작해서 `약 48번` 배열의 크기를 기존 크기의 절반만큼 늘리고 데이터를 복사하게 됩니다.

대부분의 작업은 빈공간이 있는 배열에 데이터 추가하는 작업이므로 `O(1)` 입니다. 최악의 경우만 생각해서 데이터 추가에 대한 시간복잡도를 `O(n)`으로 취급하고 데이터 추가 작업은 느리다고 판단하기에는 애매해 보입니다.

그래서 이러한 모든 경우를 고려해서 `평균 실행 시간`을 고려한 `분할 상환 분석(Amortized Analysis)`을 수행합니다.

## 분할 상환 분석

[Wikipedia](https://en.wikipedia.org/wiki/Amortized_analysis) 를 참고해보면, 작업 실행 시간의 평균을 분석함을 언급하고 있습니다.

> amortized analysis averages the running times of operations in a sequence over that sequence.  
> 분할 상환 분석은 해당 시퀀스에 대한 시퀀스의 작업 실행 시간을 평균화합니다.

한 블로그([링크](https://gazelle-and-cs.tistory.com/87))의 실생활 예제를 보면 아래와 같습니다. 내용이 길어서 간단하게 추렸습니다. 상세한 내용이 궁금하신 분은 링크를 확인하시면 됩니다.

> 회사에 커피 메이커가 있을 때, 커피가 이미 내려져있는 경우 커피를 컵에 받기 위한 시간은 `10초`, 그렇지 않은 경우 커피를 내리기 위해 `10분`이 걸립니다. 그리고 한 번 커피를 내리면 `10명`이 마실 수 있습니다.  
> 커피 메이커가 비어 있는 상황에서 `10명`이 커피를 마신다면, 첫 번째 인원이 `10분`, 나머지 인원은 `10초 x 9명 = 90초`의 시간이 필요하므로, 총 `11분 30초`의 시간이 필요합니다.  
> `11분 30초`를 10으로 나누면 평균적으로 `1.15분`의 시간이 필요합니다.

우리가 평소에 간단하게 사용하는 평균을 구하는 방법입니다. 이러한 방법을 `합계 분석(Aggregate Analysis)`라 하며, 분할 상환 분석이 등장하고서 분할 상환 분석의 한 방법으로 포함되었습니다.

### 분할 상환 분석 방법

Wikipedia에 따르면, 일반적인 분할 상환 분석 방법으로는 앞서 언급한 `합계 분석`, 합계 분석의 한 형태인 `회계법(Accounting method)`, 그리고 회계법의 한 형태인 `포텐셜법(Potential method)`이 있습니다.

합계 분석 시 상한(Upper Bound)을 _T_(_n_) 이라 할 때, amortized cost 는 _T_(_n_) / _n_ 이 됩니다. 그런데 이 _T_(_n_) 을 현실적으로 쉽게 얻을 수 없기 때문에 회계법, 포텐셜법 등을 사용하여 구합니다.

각각의 자세한 분석 방법은 각각 별도의 글을 써도 될만큼 긴 관계로 생략하도록 하겠습니다.

## Outro

상환 시간 분석을 통해 ArrayList 데이터 추가 작업에 대한 시간복잡도가 최악의 경우 `O(n)` 이니까 느려서 쓸 수 없다고 판단하는 것은 잘못된 것이라는 것을 알 수 있습니다.

하지만 단순히 ArrayList를 상환 시간 분석 해보니 평균적인 경우 `O(1)`니까 성능이 중요한 곳에서는 ArrayList 를 써야겠다고 판단하는 것도 잘못된 판단입니다.

예를 들어, HFT(High Frequency Trading)과 같이 속도가 중요한 트레이딩 프로그램을 만들면서 Java의 ArrayList를 사용했다고 가정하겠습니다. 다음에 들어올 거래 데이터를 보고 매수/매도를 판단하려는데, 하필이면 그 타이밍에 배열의 빈 공간이 없는 상태라면 배열을 복사하느라 갑자기 데이터 처리 속도가 느려저서 매수/매도 시점을 놓칠 수 있습니다.

알고리즘을 증명하거나, 다른 사람을 설득하기 위해서는 이러한 분석 방법이 중요하겠지만, 실제로 구현할 때는 상황과 가능한 입력을 고려하여 가장 적합한 알고리즘을 선택하는 것이 중요해 보입니다.

## 참고 자료

- [Wikipedia - Amortized analysis](https://en.wikipedia.org/wiki/Amortized_analysis)
- [Wikipedia - Average-case complexity](https://en.wikipedia.org/wiki/Average-case_complexity)
- [블로그 - 분할 상환 분석(Amortized Analysis)](https://gazelle-and-cs.tistory.com/87)
- [Youtube - " 저절로 되는 거 아닌가요...? " 【 자료구조 】](https://www.youtube.com/watch?v=PU_EBlEi5U8)