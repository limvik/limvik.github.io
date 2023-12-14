---
layout: post
title: 프로그래머스 Lv2. k진수에서 소수 개수 구하기
categories:
- CondingTest
- Programmers
tags:
- CodingTest
- Programmers
- Java
date: 2023-12-14 13:31 +0900
---
## Intro

[2022 KAKAO BLIND RECRUITMENT](https://tech.kakao.com/2022/01/14/2022-kakao-recruitment-round-1/) 문제 중 하나인 `k진수에서 소수 개수 구하기`([프로그래머스 링크](https://school.programmers.co.kr/learn/courses/30/lessons/92335AIA))를 풀어봤습니다.

## 문제 정보

### 문제 설명

양의 정수  `n`이 주어집니다. 이 숫자를  `k`진수로 바꿨을 때, 변환된 수 안에 아래 조건에 맞는 소수(Prime number)가 몇 개인지 알아보려 합니다.

-   `0P0`처럼 소수 양쪽에 0이 있는 경우
-   `P0`처럼 소수 오른쪽에만 0이 있고 왼쪽에는 아무것도 없는 경우
-   `0P`처럼 소수 왼쪽에만 0이 있고 오른쪽에는 아무것도 없는 경우
-   `P`처럼 소수 양쪽에 아무것도 없는 경우
-   단,  `P`는 각 자릿수에 0을 포함하지 않는 소수입니다.
    -   예를 들어, 101은  `P`가 될 수 없습니다.

예를 들어, 437674을 3진수로 바꾸면  `211`0`2`01010`11`입니다. 여기서 찾을 수 있는 조건에 맞는 소수는 왼쪽부터 순서대로 211, 2, 11이 있으며, 총 3개입니다. (211, 2, 11을  `k`진법으로 보았을 때가 아닌, 10진법으로 보았을 때 소수여야 한다는 점에 주의합니다.) 211은  `P0`  형태에서 찾을 수 있으며, 2는  `0P0`에서, 11은  `0P`에서 찾을 수 있습니다.

정수  `n`과  `k`가 매개변수로 주어집니다.  `n`을  `k`진수로 바꿨을 때, 변환된 수 안에서 찾을 수 있는  **위 조건에 맞는 소수**의 개수를 return 하도록 solution 함수를 완성해 주세요.

### 제한사항

-   1 ≤  `n`  ≤ 1,000,000
-   3 ≤  `k`  ≤ 10

### 입출력 예

|n|k|result|
|--|--|--|
|437674|3|3|
|110011|10|2|

## 풀이 계획

앞서 풀었던 호텔 대실 문제([링크](/posts/programmers-booking-hotel-room-155651/))에서 급한 마음에 계획도 없이 냅다 구현하다가 시간을 초과해서, 이번에는 일단 계획부터 세웠습니다.

1. n을 k 진수로 변환하고, 계산 편의를 위해 마지막에 0 추가
2. 변환한 숫자에서 첫번째 0이 위치한 index를 탐색
3. 0번 index부터 첫 0이 위치한 index 전까지의 숫자가 소수인지 판별
4. 첫 0이 위치한 index를 0번을 가리키던 index 변수에 대입
5. 다음 0이 위치한 index 탐색하여 0과 0 사이의 숫자가 소수인지 판별 하는 것을 반복

첫번째 입출력 예를 가지고 보면, 아래와 같습니다.

![01.풀이 계획 예시](/assets/img/2023-12-14-programmers-count-prime-number-in-base-number-k-92335AIA/01.solving-plan-example.png)

그리고 키보드에 손을 올리자마자, [`split()`]((https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html#split%28java.lang.String%29)) method를 사용하면 되는데 왜 그랬지? 하는 생각과 함께 바로 `split()`을 사용하는 것으로 계획을 변경했습니다.

## 코드

### 1차 풀이

일단 코드 설명은 뒤에서 하기로 하기로 하고, 여기서는 runtime 오류가 발생합니다.

`Integer.parseInt(num)`을 했을 때, Integer의 범위를 초과하기 때문입니다.

예를들어, 범위 내 숫자 중 `997,271` 은 3진수로 변환 시 `1,212,122,222,222`이 됩니다.

```java
class Solution {
    public int solution(int n, int k) {
        int answer = 0;
        
        String convertedNum = Integer.toString(n, k);
        String[] nums = convertedNum.split("0+");
        for (String num : nums) {
            if (isPrime(Integer.parseInt(num))) {
                ++answer;
            }
        }
        
        return answer;
    }
    
    private boolean isPrime(int n) {
        if (n <= 1) return false;
        else if (n <= 3) return true;
        
        for (int i = 3; i < (int) Math.sqrt(n); i+=2) {
            if (n % i == 0) return false;
        }
        return true;
    }
}
```

### 2차 풀이

범위 초과 문제를 해결하기 위해 `Long` 으로 전부 변환합니다. 그러면, 답이 틀리는 문제가 있습니다. `isPrime()` method에서 `4`를 소수로 판별하기 때문입니다.

이전에 소수 판별 문제가 자주 보여서 Loop 부분만 외워서 사용하다가, 4를 놓치는 실수를 했습니다.

```java
class Solution {
    public int solution(int n, int k) {
        int answer = 0;
        
        String convertedNum = Integer.toString(n, k);
        String[] nums = convertedNum.split("0+");
        for (String num : nums) {
            if (isPrime(Long.parseLong(num))) {
                ++answer;
            }
        }
        
        return answer;
    }
    
    private boolean isPrime(long n) {
        if (n <= 1) return false;
        else if (n <= 3) return true;
        
        for (long i = 3; i < (long) Math.sqrt(n); i+=2) {
            if (n % i == 0) return false;
        }
        return true;
    }
}
```

### 3차 풀이

4까지 고려해서 소수 판별을 수행하도록 수정해서 통과합니다.

```java
class Solution {
    public int solution(int n, int k) {
        int answer = 0;

        String convertedNum = Integer.toString(n, k);
        String[] nums = convertedNum.split("0+");
        for (String num : nums) {
            if (isPrime(Long.parseLong(num))) {
                ++answer;
            }
        }

        return answer;
    }

    private boolean isPrime(long n) {
        if (n <= 1 || n == 4) return false;
        else if (n <= 3) return true;

        for (long i = 3; i <= (long) Math.sqrt(n); i+=2) {
            if (n % i == 0) return false;
        }
        return true;
    }
}
```

뒤에서 글을 쓰다 알아차렸지만 여기에도 오류가 있습니다. 코드를 차례대로 살펴보며 설명드리겠습니다.

#### solution method

##### 진수 변환

`Integer`나 `Long` 클래스의 `toString()` method의 두 번째 인수에 변환할 진수(radix)를 넣어서 변환할 수 있습니다.

```java
String convertedNum = Integer.toString(n, k);
```

처음에 int로 해서 `Integer`로 하고 변경을 안했는데, [`Long`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Long.html)에도 동일한 method가 있습니다.

##### 숫자 추출

그리고 0을 기준으로 각 숫자를 나누어 추출하고, 문자열 배열로 변환합니다.

`split()` method의 인수로는 정규표현식을 사용하기 때문에, 기준인 0이 1개 이상이라는 의미에서 `+`를 붙여줍니다.

```java
String[] nums = convertedNum.split("0+");
```

##### 소수 판별 요청

변환한 문자열 배열에서 하나씩 숫자를 가져와 `Long` type으로 변환하고, `isPrime()` method에 소수 판별을 요청합니다. 소수가 맞다면 `반환할 값을 1 증가`시킵니다.

```java
for (String num : nums) {
    if (isPrime(Long.parseLong(num))) {
        ++answer;
    }
}
```

#### isPrime method

다음은 소수를 판별하는 `isPrime()` method 입니다.

```java
private boolean isPrime(long n) {
    if (n <= 1 || n == 4) return false;
    else if (n <= 3) return true;

    for (long i = 3; i <= (long) Math.sqrt(n); i+=2) {
        if (n % i == 0) return false;
    }
    return true;
}
```

먼저 for Loop 부분을 살펴보겠습니다.

`long i = 3` 으로 3부터 시작했는데, 어차피 짝수는 앞에서 판별을 해야되는데... 안했는데도 통과가 됐었네...? 어라? 그러면... 2는 소수니까 2판별 후에 짝수를 검사하는 로직을 추가합니다.

```java
private boolean isPrime(long n) {
    if (n <= 1) return false;
    else if (n <= 3) return true;
    else if (n % 2 == 0) return false;
    
    for (long i = 3; i <= (long) Math.sqrt(n); i+=2) {
        if (n % i == 0) return false;
    }
    return true;
}
```

여러 테스트 케이스 중에 2를 초과하는 짝수가 4 밖에 없다는게 신기합니다.

그럼 이제 진짜 짝수는 이미 판별을 했으니 홀수만 검사하면 됩니다. 그런데 1이 아닌 이유는 당연히 아시겠지만, 1로 나누면 무조건 나누어 떨어져 소수가 아닌 것이 되기때문에 앞에서 판별합니다.

결과적으로 초기값은 3으로 시작해서 2씩 증가하며 홀수로만 나누어 떨어지는지 확인하고, 나누어 떨어진다면 소수가 아닌 것으로 판단합니다.

다음으로 for Loop의 비교식을 살펴보겠습니다.

```java
i <= (long) Math.sqrt(n)
```

어떤 숫자의 제곱근보다 큰 정수를 곱해서는 해당 숫자를 구할 수 없습니다. 예를 들어 4의 경우 sqrt(4) = 2이고 2 x 2 = 4 가 될 수 있지만, 3에는 어떤 정수를 곱해도 4가 될 수 없습니다.

그래서 위 예에서 처럼 제곱근인 2까지는 가능성이 있으므로, `<=` 로 판단을 하고, `Math.sqrt()`의 반환 값은 실수이기 때문에, 정수로 변환합니다.

Loop 순회 중 홀수로 나누어 떨어지는 것이 있다면, 소수가 아님을 반환합니다.

```java
if (n % i == 0) return false;
```

마지막으로 Loop를 다 돌았는데도, 나누어떨어지는게 없다면 소수이므로 true를 반환합니다.

## Outro

누군가의 추천으로 문제 풀면서 사고과정을 훑어보려고 썼는데, 짝수 판별을 안했던 것과 같은 구멍도 찾을 수 있었습니다.

여전히 Corner Case를 잘 생각하지 못하는 문제가 보입니다. 

상한인 `1,000,000` 을 다른 진수로 변환했을 때, 몇 자리수나 나오는지 확인해 봤으면 좋았을 텐데 아쉽습니다.
