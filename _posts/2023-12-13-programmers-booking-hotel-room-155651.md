---
layout: post
title: 프로그래머스 Lv2. 호텔 대실
categories:
- CodingTest
- Programmers
tags:
- CodingTest
- Programmers
- Java
date: 2023-12-13 15:24 +0900
---
## Intro

프로그래머스에서 Lv2. 문제인 호텔 대실 문제([링크](https://school.programmers.co.kr/learn/courses/30/lessons/155651))를 풀어보았습니다. 어이 없는 실수를 많이해서 정리 겸 작성합니다.

## 문제 정보

### 문제 설명

호텔을 운영 중인 코니는 최소한의 객실만을 사용하여 예약 손님들을 받으려고 합니다. 한 번 사용한 객실은 퇴실 시간을 기준으로 10분간 청소를 하고 다음 손님들이 사용할 수 있습니다.  
예약 시각이 문자열 형태로 담긴 2차원 배열 `book_time`이 매개변수로 주어질 때, 코니에게 필요한 최소 객실의 수를 return 하는 solution 함수를 완성해주세요. 

### 제한사항

-   1 ≤  `book_time`의 길이 ≤ 1,000
    -   `book_time[i]`는 ["HH:MM", "HH:MM"]의 형태로 이루어진 배열입니다
        -   [대실 시작 시각, 대실 종료 시각] 형태입니다.
    -   시각은 HH:MM 형태로 24시간 표기법을 따르며, "00:00" 부터 "23:59" 까지로 주어집니다.
        -   예약 시각이 자정을 넘어가는 경우는 없습니다.
        -   시작 시각은 항상 종료 시각보다 빠릅니다.

### 입출력 예

| book_time | result |
|--|--|
| [["15:00", "17:00"], ["16:40", "18:20"], ["14:20", "15:20"], ["14:10", "19:20"], ["18:20", "21:20"]] | 3 |
| [["09:10", "10:10"], ["10:20", "12:20"]] | 1 |
| [["10:20", "12:30"], ["10:20", "12:30"], ["10:20", "12:30"]] | 3 |

## 풀이 계획

급하게 풀 마음에 풀이 계획이고 뭐고, 주어진 예약 정보를 빈 시간에 잘 채워넣으면 되겠네(?) 라는 생각만 갖고 풀기 시작했습니다. 그러다 상한 시간으로 잡은 1시간을 초과했습니다.

## 추가 테스트 케이스

이 글을 보신다면, 안 풀려서 보시는 분일 가능성이 높으니, 한번 더 살펴보시라고 추가적인 테스트 케이스를 먼저 드립니다.

아래 테스트 케이스로 정렬은 했는지, 청소 시간을 고려했을 때 종료 시간이 자정을 넘어 간 경우는 고려했는지 테스트 해볼 수 있습니다.

| 목적 | book_time | result |
|--|--|--|
| 정렬 여부 확인 | [["18:20", "21:20"], ["14:20", "15:20"], ["15:00", "17:00"], ["16:40", "18:20"], ["14:10", "19:20"]] | 3 |
| 청소 시간으로 자정이 넘어 가는 경우 확인 | [["00:05", "23:59"], ["00:10", "00:20"]] | 2 |

## 코드

### 전체 코드

먼저 전체 코드는 아래와 같습니다. ArrayList를 이용하였습니다.

```java
import java.util.*;
import java.time.*;

class Solution {
    
    private static final long CLEANING_TIME = 10L;
    
    public int solution(String[][] book_time) {
        
        List<Book> requestedBooks = new ArrayList<>(book_time.length * book_time[0].length);
        for (int i = 0; i < book_time.length; i++) {
            var startTime = LocalTime.parse(book_time[i][0]);
            var endTime = LocalTime.parse(book_time[i][1]);
            requestedBooks.add(new Book(startTime, endTime));
        }
        
        Collections.sort(requestedBooks);
        
        List<Room> rooms = new ArrayList<>();
        rooms.add(new Room());
        
        for (var requestedBook : requestedBooks) {
            var isBooked = false;
            for (var room : rooms) {
                if (room.bookIfAbsent(requestedBook) != null) {
                    isBooked = true;
                    break;
                }
            }
            
            if (!isBooked) {
                var newRoom = new Room();
                newRoom.bookIfAbsent(requestedBook);
                rooms.add(newRoom);
            }
        }
        
        return rooms.size();
    }
    
    class Room {
        
        private List<Book> books = new ArrayList<>();
        
        public Book bookIfAbsent(Book requestedBook) {
            // 중복된 예약이 있는 경우 null 반환
            if (books.size() > 0 && ! books.get(books.size()-1).isNotDuplicated(requestedBook)) {
                return null;
            }
            
            books.add(requestedBook);
            return requestedBook;
        }
        
    }
    
    class Book implements Comparable<Book> {
        
        private LocalTime startTime;
        private LocalTime endTime;
        
        public Book(LocalTime startTime, LocalTime endTime) {
            this.startTime = startTime;
            this.endTime = endTime;
        }
        
        public LocalTime getStartTime() {
            return startTime;
        }
        
        public LocalTime getEndTime() {
            return endTime;
        }
        
        public boolean isNotDuplicated(Book other) {
            if (endTime.isAfter(LocalTime.parse("23:48"))) {
                return false;
            }
            return endTime.plusMinutes(CLEANING_TIME - 1).isBefore(other.getStartTime());
        }
        
        @Override
        public int compareTo(Book other) {
            int comparedValue = startTime.compareTo(other.getStartTime());
            if (comparedValue == 0) {
                comparedValue = endTime.compareTo(other.getEndTime());
            }
            return comparedValue;
        }
        
    }
}
```

### solution method

먼저 solution method 내에 있는 코드부터 설명 드리겠습니다.

#### 입력인 book_time 2차원 배열을 Book 객체의 ArrayList로 변환 및 정렬

```java
List<Book> requestedBooks = new ArrayList<>(book_time.length * book_time[0].length);
for (int i = 0; i < book_time.length; i++) {
    var startTime = LocalTime.parse(book_time[i][0]);
    var endTime = LocalTime.parse(book_time[i][1]);
    requestedBooks.add(new Book(startTime, endTime));
}

Collections.sort(requestedBooks);
```

`PriorityQueue` 를 사용하신 분들도 보이는데, PriorityQueue는 element를 넣을 때 마다 비교하고 정렬해서 넣다보니 개인적으로 비효율적이라 생각되어 `ArrayList`로 element를 모두 추가한 후 정렬하기로 하였습니다.

입력 데이터 자체가 많지는 않아서 유의미한 차이는 없어보입니다. 오른쪽이 PriorityQueue를 사용한 채점 결과입니다.

![ArrayList와 PriorityQueue를 사용한 비교](/assets/img/2023-12-13-programmers-booking-hotel-room-155651/01.array-list-vs-priority-queue-result.png)

그리고 ArrayList의 size는 명확하다보니, ArrayList Size 증가를 위해 시간이 지연되는 것을 방지하고자 `book_time.length * book_time[0].length` 로 size를 지정하였습니다. 마찬가지로 데이터 갯수가 최대 1000개라 크게 신경써야 할 부분은 아닌 것 같습니다.

다음으로 예약 정보를 저장하기 위해서 `LocalTime` 을 이용해 문자열 시간 정보를 parsing 한 후 `Book` 객체로 만들어 앞서 정의한 ArrayList 에 저장하였습니다. `Book` 객체는 나중에 뒤에서 살펴보겠습니다.

입력인 `book_time`의 모든 데이터를 `Book` 객체의 ArrayList로 변환한 후 `Collections.sort(requestedBooks);`를 통해 정렬합니다. 아무 순서로나 Booking을 할 경우, 각 방마다 비어있는 예약 시간을 확인하고 최적의 장소에 배치해야 하므로, 미리 `시작 시간(startTime)을 기준으로 정렬`합니다. 시작 시간을 기준으로 정렬해야 Room의 비어있는 앞쪽 공간부터  채워나갈 수 있어 종료 시간(endTime)만 확인하고도 Booking 가능 여부를 판단할 수 있습니다.

#### requestedBooks 를 기존 예약 정보가 있는 Room에 추가하거나 새로운 Room에 추가

이어서 solution method에 있는 나머지 코드를 보겠습니다.

```java
List<Room> rooms = new ArrayList<>();
rooms.add(new Room());

for (var requestedBook : requestedBooks) {
    var isBooked = false;
    for (var room : rooms) {
        if (room.bookIfAbsent(requestedBook) != null) {
            isBooked = true;
            break;
        }
    }
    
    if (!isBooked) {
        var newRoom = new Room();
        newRoom.bookIfAbsent(requestedBook);
        rooms.add(newRoom);
    }
}

return rooms.size();
```

`Room` 객체의 ArrayList를 만듭니다. 그리고 제한사항에 `book_time`이 최소 1개 있다고 하였으므로, 기본적으로 최소 1개의 `Room`이 필요하므로 바로 추가(`rooms.add(new Room());`)합니다.

다음으로 앞서 정렬한 `Book` 객체의 ArrayList인 requestedBooks에 enhanced for문을 이용하여 `Book` 객체를 하나씩 가져와 현재 Booking 정보가 있는 `Room` 중에 여유 시간이 있는 `Room`이 있는지 확인하고, 추가할 수 있으면 `Book` 객체를 추가합니다.

만약 `Book` 객체를 추가할 수 없으면, 새로운 `Room`을 만들어 `Book` 객체를 추가합니다.

마지막으로 ArrayList에 있는 총 `Room` 객체의 갯수가 필요한 `Room`의 최소 갯수이므로, rooms 의 size를 답으로 반환합니다.

### Room class

다음으로 예약 정보인 `Book` 객체를 저장하고, 관리하는 `Room` class 입니다.

```java
class Room {
    
    private List<Book> books = new ArrayList<>();
    
    public Book bookIfAbsent(Book requestedBook) {
        // 중복된 예약이 있는 경우 null 반환
        if (books.size() > 0 && ! books.get(books.size()-1).isNotDuplicated(requestedBook)) {
            return null;
        }
        
        books.add(requestedBook);
        return requestedBook;
    }
    
}
```

`Room`에 대한 예약 정보인 `Book` 객체를 저장할 ArrayList를 추가하였습니다.

그리고 시간이 중복되는 Booking 정보가 없다면, `Book` 객체를 추가할 수 있도록 `bookIfAbsent()` method를 추가하였습니다. `Room` 객체에 `Book` 객체가 추가될 때는 `시작 시간이 가장 빠른 것`이 추가되므로, 앞쪽의 Booking 정보와는 비교할 필요가 없습니다. 그래서 `books.get(books.size()-1)` 을 통해 마지막 `Book` 객체만 가져와 `isNotDuplicated()` method를 통해 새로 Booking 요청된 정보인 `requestedBook`이 중복된 Booking 정보인지 판단합니다.

만약 기존 Booking 정보와 겹친다면 null을 반환하고, 겹치는 정보가 아니라면, `Room` 객체가 관리하는 `Book` 객체의 ArrayList에 추가한 후 client에 요청한 Booking 정보를 반환합니다.

### Book class

이미 앞서 많이 보았던 Booking 정보를 저장하는 `Book` class 입니다.

```java
class Book implements Comparable<Book> {
        
        private LocalTime startTime;
        private LocalTime endTime;
        
        public Book(LocalTime startTime, LocalTime endTime) {
            this.startTime = startTime;
            this.endTime = endTime;
        }
        
        public LocalTime getStartTime() {
            return startTime;
        }
        
        public LocalTime getEndTime() {
            return endTime;
        }
        
        public boolean isNotDuplicated(Book other) {
            if (endTime.isAfter(LocalTime.parse("23:48"))) {
                return false;
            }
            return endTime.plusMinutes(CLEANING_TIME - 1).isBefore(other.getStartTime());
        }
        
        @Override
        public int compareTo(Book other) {
            int comparedValue = startTime.compareTo(other.getStartTime());
            if (comparedValue == 0) {
                comparedValue = endTime.compareTo(other.getEndTime());
            }
            return comparedValue;
        }
        
    }
```

정렬 시에 사용할 수 있도록 `Comparable` interface를 상속받아 `compareTo` 메서드를 override하였습니다. 시작 시간(startTime)을 첫 번째 기준으로 정렬하고, 시작 시간이 같을 경우 종료 시간(endTime)을 기준으로 정렬합니다. 이를 통해 정렬이 되면, 시작 시간이 빠른 `Book` 객체가 `Room`에 먼저 추가되어 `Room` 의 모든 `Book` 객체를 확인하지 않아도 되게합니다.

Booking 정보인 시작 시간과 종료 시간의 자료형은 모두 `LocalTime`을 사용하여, `hh:mm` 형식의 시간을 조작하기 편하게 하였습니다.

getter 메서드는 설명 안해도 아실테고...

저를 괴롭혔던 `isNotDuplicated()` method를 보겠습니다. `isDuplicated()`로 해도 됐을텐데, 처음에 풀었던 방식이 `isDuplicated()`로 하면 비교해야 할 것이 많아 `isNotDuplicated()`로 했었습니다. 그리고 그 흔적이 그대로 남아버렸습니다.

보기 불편하니까 한번 더 코드를 붙여넣겠습니다.

```java
public boolean isNotDuplicated(Book other) {
    if (endTime.isAfter(LocalTime.parse("23:48"))) {
        return false;
    }
    return endTime.plusMinutes(CLEANING_TIME - 1).isBefore(other.getStartTime());
}
```

종료 시간이 `23:48` 이후일 때 중복인 것으로 판단한 이유는 청소 시간 10분을 고려했을 때, 다른 사람이 Booking할 수 있는 시각인 `23:58 ~ 23:59` 를 고려하기 위함입니다. `isAfter` 이므로 `23:58 ~ 23:59` 이 Booking 가능한지 판단하기 위해 기존 `Book` 종료 시간이 `23:48` 일때는 `청소 시간 9분`을 추가하여 `23:57` 을 만들어 판단합니다. 현실적으로는 이런 케이스는 없겠죠.

제일 중요한 목적은 `23:49` 부터는 청소 시간을 고려했을 때, 모든 시간이 중복으로 판단되어야 하기 때문입니다. 이를 고려하지 않으면 `23:51` 부터는 9를 더했을 때 `00:00`이 되어, 다른 Booking 정보의 시작 시간이 `00:01` 이후인 경우 중복되지 않았다고 판단할 가능성이 있습니다.

이외의 경우에는 종료 시간에 청소 시간인 10분을 고려하기 위해 9분을 더했을 때도, 종료 시간이 Booking 하고자하는 시간의 시작 시간보다 앞에 있는지 비교한 결과를 boolean 값으로 반환합니다.

참고로, 10분이 아닌 9분을 더한 이유는 `isBefore`이니까 `Equal`인 경우는 고려하지 않기 때문입니다. 예를들어, 10:10에 청소가 끝나면, 시작 시간이 10:10인 고객은 Booking을 할 수 있어야 하는데, 분 단위로 데이터를 관리하므로 9분을 더하면 `Equal`인 경우까지 고려할 수 있습니다.

### 시간복잡도

정렬할 때 O(n log n), 각 `Book` 마다 `Room`이 필요한 경우 `Book` ArrayList를 모두 순회하면서 `Room`도 모두 순회하므로 O(n^2) 입니다.

O(n log n + n^2) 이므로, `O(n^2)`이 됩니다.

## 잘못한 것들

1. 급한 마음에 계획도 없이 그냥 품
2. corner case 에 대해 생각 못함

코드에서는 수정해서 적지 않았지만, 가장 오래 걸린 문제는 기존에 잘못된 코드를 수정하면서 남겨두었던 잘못된 Booking 정보 `중복 여부 판단 로직`이었습니다. 마지막 종료 시간과 새로운 Booking 정보의 시작 시간과만 비교하면 되는데, 마지막 시작 시간이 새로운 Booking 정보의 종료 시간보다 뒤에 있는지도 확인했습니다.

이로 인해서 앞쪽 시간은 이미 다 Booking이 되어 있는데도 불구하고, 중복되지 않았다고 판단하는 경우가 생겼습니다.

이 문제도 계획없이 풀었던것이 원인이 된 것으로 보입니다. 심리적인 부분의 개선이 제일 급한 것 같습니다. 연습만이 살 길?

## Outro

결론, 생각좀 하고 풀자.
