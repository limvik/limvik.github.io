---
layout: post
title: SWEA 2805 농작물 수확하기
categories:
- CodingTest
- SWEA
tags:
- CodingTest
- SWEA
- Java
date: 2023-04-07 14:50 +0900
---
## Intro
수업 시간에 코딩 테스트 문제 풀었던 것 중에 몇가지는 기록으로 남기면 좋을 것 같아서 일단 시도해 봅니다.

## 문제
문제는 수업 시간에 사용하는 SW Expert Academy의 `농작물 수확하기`([링크](https://swexpertacademy.com/main/code/problem/problemDetail.do?contestProbId=AV7GLXqKAWYDFAXB)) 문제입니다.

자세한 문제는 링크에 나와있고, 간단하게 보자면 아래와 같이 입력으로 제시되는 정수 배열에서 파란색으로 칠해진 곳의 값을 모두 더해서 출력하는 문제입니다.

![SWEA 2805](/assets/img/2023-04-07-coding-test-swea-2805/2023-04-07-farm-array.png)

## 해결 전략

제가 찾은 해결 전략은 가장 윗 줄부터 아래줄로 이동할 때마다 열의 중간을 기준으로 열의 좌우가 각각 한 칸씩 넓어지거나 좁아지는 특성을 이용하는 것이었습니다.

![SWEA 2805](/assets/img/2023-04-07-coding-test-swea-2805/2023-04-07-solving-strategy.png)

배열이라 가정하고, 행 인덱스를 0부터 증가시키고, 기준값에서 행 인덱스만큼 뺀 것을 열 이동 시 시작지점, 그리고 기준값에서 행 인덱스만큼 더한 것을 열 이동 시 종료지점으로 설정한 for문을 이용하였습니다.

예시로, 행 인덱스가 2일 때 시작지점과 종료 지점을 표시해봤습니다.

![SWEA 2805](/assets/img/2023-04-07-coding-test-swea-2805/2023-04-07-solving-strategy-example.png)

그런데 행의 중앙을 기준으로 다시 줄어들 때를 어떻게 처리해야 할지 떠오르지 않아서 위 아래로 나누어서 처리를 했습니다.

![SWEA 2805](/assets/img/2023-04-07-coding-test-swea-2805/2023-04-07-solving-strategy-divide.png)

그리고 아래와 같이 코드를 작성했습니다. (주요 알고리즘만 추가했습니다.)
```java
// 수익 계산
int answer = 0;
int criteria = farmSize/2; // 중간 인덱스값
// 행 기준 중앙까지 계산 i가 증가하면서 좌우 범위가 늘어남
for (int i = 0; i <= criteria; ++i) {
	char[] profits = stdIn.nextLine().toCharArray();
	for (int j = criteria - i; j <= criteria + i; ++j) {
		answer += Character.getNumericValue(profits[j]);
	}
}

// 나머지 계산
// i가 감소하면서 좌우 범위가 줄어듬
for (int i = criteria-1; i >= 0; --i) {
	char[] profits = stdIn.nextLine().toCharArray();
	for (int j = criteria - i; j <= criteria + i; ++j) {
		answer += Character.getNumericValue(profits[j]);
	}
}
```
참고로 입력은 아래와 같이 첫째 줄은 테스트 케이스 갯수, 둘째 줄은 배열의 크기, 나머지는 배열 원소 입니다.
```
1
5
14054
44250
02032
51204
52212
```
답은 나오지만 뭔가 마음에 들지 않습니다.

## 조금 더 개선해 보기

결국 모두 탐색해야 하므로, 효율을 개선할 방법은 찾기 어렵지만 for 문을 둘로 나눠서 돌린게 너무 마음에 안들었습니다. 중복 코드는 메서드로 만들어버린다고 쳐도 for 문을 어떻게 합칠 것인가... 고민을 하다가 시간이 너무 오래 걸려서 물어봤습니다. chatGPT한테... 근데 틀린 답을 주더군요.

```java
for (int i = 0; i < farmSize; ++i) {
    String profits = stdIn.nextLine();
    int diff = Math.abs(criteria - i);
    for (int j = criteria - diff; j <= criteria + diff; ++j) {
        answer += Character.getNumericValue(profits.charAt(j));
    }
}
```
그럴듯해 보이지만 i가 0일때(=첫 번째 줄), j가 0이라 모든 행의 값을 더해버립니다. 그래도 `쓸데없이 char 배열로 변환했던 거는 잘 짚어줬습니다`.

그리고 별도의 차이값을 저장하는 `변수를 사용하는 아이디어`도 얻었겠다. N = 5일 때, 0 1 2 1 0 과 같은 수열을 얻을 수 있는 공식을 구해서 수정해 봤습니다.

```java
for (int i = 0; i < farmSize; ++i) {
    String profits = stdIn.nextLine();
    int diff = criteria - Math.abs(criteria - i);
    for (int j = criteria - diff; j <= criteria + diff; ++j) {
        answer += Character.getNumericValue(profits.charAt(j));
    }
}
```
수학 잘하고 싶어집니다.

## Outro

수학적인 능력을 단기간에 올리기는 어려울테니, 떠오른 생각을 빠르게 검증해본 후에 개선하는 방식을 취해야겠습니다. 매번 처음떠오른 방법으로 풀다가 '더 좋은 방법이 있을 것 같은데...' 하면서 시간을 많이 잡아먹는게 실전에서는 문제가 될 것 같습니다.

개선할 것을 정리하자면,
- char 배열이 꼭 필요한게 아니면, String charAt() 메서드 사용하기
- 떠오른 해결방법은 빠르게 검증해서 풀이시간 줄이기
- 추가적인 저장공간을 활용해서 개선할 수 있는지 생각해보기

이정도가 될 것 같습니다. 다음에 또 다시 보면서 놓친게 있는지 봐야겠습니다.
