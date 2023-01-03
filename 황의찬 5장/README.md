# 5장 - 스트림 활용
다음은 데이터 컬렉션 반복을 명시적으로 관리하는 외부 반복 코드입니다.  
```java
 List<Dish> vegetarianDishes = new ArrayList<>();
    for (Dish d : menu) {
      if (d.isVegetarian()){
        vegetarianDishes.add(d);
      }
    }
```
명시적 반복 대신 filter와 collect 연산을 지원하는 스트림 API를 이용해서 데이터 반복을 내부적으로 처리할 수 있습니다.  
```java
List<Dish> vegetarianDishesWithStream = menu.stream()
        .filter(Dish::isVegetarian)
        .collect(Collectors.toList());
```
데이터를 어떻게 처리할지는 스트림 API가 관리하므로 편리하게 데이터 관련 작업을 할 수 있습니다.  
따라서 스트림 API 내부적으로 다양한 최적화가 이루어질 수 있습니다. 스트림 API는 내부 반복 뿐 아니라 코드를 병렬로 실행할지 여부도 결정할 수 있습니다.  
이러한 일은 순차적인 반복을 단일 스레드로 구현하는 외부 반복으로는 달성할 수 없습니다.  
  
## 5.1 필터링 
5.1절에서는 스트림의 요소를 선택하는 방법, 프레디케이트 필터링 방법과 고유 요소만 필터링하는 방법을 배웁니다.  
### 5.1.1 프레디케이트로 필터링
스트림 인터페이스는 filter 메서드를 지원합니다.  
filter 메서드는 **프레디케이트(불리언을 반환하는 함수)**를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환합니다.  
예를 들어 모든 채식요리를 필터링해서 채식 메뉴를 만들 수 있습니다.  
```java
List<Dish> vegetarianMenu = menu.stream()
    .filter(Dish::isVegetarian)
    .collect(Collectors.toList());
```

### 5.1.2 고유 요소 필터링 
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원합니다.(고유 여부는 스트림에서 만든 객체의 hashCode, equals)로 결정됩니다.  
다음 코드는 리스트의 모든 짝수를 선택하고 중복을 필터링합니다.  
```java
 List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
//        .forEach(i -> System.out.printf("%d ", i));
        .forEach(Filtering::printWithSpace);
```
## 5.2 스트림 슬라이싱(Java 9부터)
5.2절에서는 스트림의 요소를 선택하거나 스킵하는 다양한 방법을 설명합니다.  
프레디케이트를 이용하는 방법, 스트림의 처음 몇 개의 요소를 무시하는 방법, 특정 크기로 스트림을 줄이는 방법 등 다양한 방법을 이용할 수 있습니다.  
  
### 5.2.1 프레디케이트를 이용한 슬라이싱 
Java9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메서드를 지원합니다.  
320 칼로리 이하의 요리를 선택하는 방법은 다음과 같이 filter를 이용하는 방법을 생각해볼 수 있다.  
```java
List<Dish> filteredMenu = specialMenu.stream()
    .filter(dish -> dish.getCalories() < 320)
    .collect(Collectors.toList());
```
위 리스트는 이미 칼로리 순으로 정렬되어 있습니다.filter 연산을 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하게 됩니다.  
따라서 리스트가 이미 정렬되어 있다는 사실을 이용해 320칼로리보다 크거나 같은 요리가 나왔을 때 (칼로리 >= 320) 반복 작업을 중단할 수 있습니다.  
  
작은 리스트에는 이와 같은 동작이 별거 아닌 것처럼 보일 수 있지만, 아주 많은 요소를 포함하는 큰 스트림에서는 상당한 차이가 될 수 있습니다.  
`takeWhile` 연산을 이용하면 이를 간단하게 처리할 수 있습니다.  
  
`takeWhile`을 이용하면 무한스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스할 수 있습니다.  
```java
List<Dish> slicedMenu1 = specialMenu.stream()
    .takeWhile(dish -> dish.getCalories() < 320)
    .collect(Collectors.toList());
```
`filter`는 조건에 대해 다 검사하며 참인것만 다음으로 넘어가지만 `takeWhile`은 조건에 대해 참이 아닐경우  
바로 거기서 멈추게 됩니다.  

나머지 요소를 선택하려면 `dropWhile`을 이용하면 된다.  
```java
List<Dish> slicedMenu2 = specialMenu.stream()
    .dropWhile(dish -> dish.getCalories() < 320)
    .collect(Collectors.toList());
```
`dropWhile`은 `takeWhile`과 정반대의 작업을 수행합니다. dropWhile은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버립니다.  
프레디케이트가 거짓이 되면 그 지점에서 작업을 중단하고 남은 모든 요소를 반환합니다.  
dropWhile은 무한한 남은 요소를 가진 무한 스트림에서도 동작합니다.  
  
### 5.2.2 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 `limit(n)` 메서드를 지원합니다.  
```java
List<Dish> dishes = specialMenu.stream()
    .filter(dish -> dish.getCalories() > 300)
    .limit(3)
    .collect(Collectors.toList());
```
예를 들어 300칼로리 이상의 세 요리를 선택해서 리스트를 만들 수 있습니다.  
filter와 limit을 조합한 모습으로, 프레디케이트와 일치하는 처음 세 요소를 선택한 다음에 즉시 결과를 반환합니다.  
  
정렬되지 않은 스트림(예를 들면 소스가 Set)에도 limit을 사용할 수 있습니다. 소스가 정렬되어 있지 않았다면 limit의 결과도 정렬되지 않은 상태로 반환된다.  
  
### 5.2.3 요소 건너뛰기
스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원합니다. n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환됩니다.  
```java
List<Dish> skip = specialMenu.stream()
    .filter(d -> d.getCalories() > 300)
    .skip(2)
    .collect(Collectors.toList());
```
예를 들어 다음 코드는 300칼로리 이상의 처음 두 요리를 건너뛴 다음에 300칼로리가 넘는 나머지 요리를 반환합니다.  

## 5.3 매핑
특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산입니다.  
예를 들어 SQL의 테이블에서 특정 열만 선택할 수 있습니다.  
스트림 API의 map과 flatMap 메서드는 `특정 데이터를 선택하는 기능`을 제공합니다.  
  
### 5.3.1 스트림의 각 요소에 함수 적용하기
스트림은 함수를 인수로 받는 map 메서드를 지원합니다. 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑됩니다.  
이 과정은 기존의 값을 고친다라는 개념보다는 새로운 버전을 만든다 라는 개념에 가까우므로 변환(transforming)에 가까운 매핑(mapping)이라는 단어를 사용합니다.  
  
```java
List<String> dishNames = menu.stream()
        .map(Dish::getName)//인수로 제공된 함수는 각 요소에 적용
        .collect(Collectors.toList());
```
getName은 문자열을 반환하므로 map 메서드의 출력 스트림은 `Stream<String>` 형식을 가집니다.  
  
단어 리스트가 주어졌을 때 각 단어가 포함하는 글자 수의 리스트를 반환한다고 가정합니다.  
리스트의 각 요소에 함수를 적용하면 가능합니다. 이전 예제에서 확인했던 것처럼 map을 이용할 수 있습니다.  
`String::length`를 map에 전달해서 문제를 해결할 수 있습니다.  
```java
 List<String> words = Arrays.asList("Modern", "Java", "In", "Action");
    List<Integer> wordLengths = words.stream()
        .map(String::length)
        .collect(Collectors.toList());
```
각 요리명의 길이를 알고 싶다면 다른 map 메서드를 연결할 수 있습니다.  
```java
List<Integer> dishNameLengths = menu.stream()
        .map(Dish::getName)
        .map(String::length)
        .collect(Collectors.toList());
```

### 5.3.2 스트림 평면화 
리스트에서 **고유 문자**로 이루어진 리스트를 반환해보겠습니다.  
```java
stringList.stream()
        .map(word -> word.split(""))
        .distinct()
        .collect(Collectors.toList());
```
위 코드에서 map으로 전달한 람다는 각 단어의 String[](문자열 배열)을 반환한다는 점이 문제입니다.  
따라서 map 메서드가 반환한 스트림의 형식은 `Stream<String[]>`입니다. 원하는 것은 `Stream<String>`입니다.  
  
다행히 flatMap이라는 메서드를 이용해서 이 문제를 해결할 수 있습니다.  
  
### map과 Arrays.stream 활용 
우선 배열 스트림 대신 문자열 스트림이 필요합니다. 다음 코드에서 보여주는 것처럼 문자열을 받아 스트림을 만드는 `Arrays.stream()` 메서드가 있습니다.  
```java
String[] arrayOfWords = {"Goodbye", "World"};
Stream<String> streamOfWords = Arrays.stream(arrayOfWords);
```
위 예제의 파이프라인에 Arrays.stream() 메서드를 적용해봅시다.  
```java
 List<Stream<String>> collect = words.stream()
        .map(word -> word.split(""))
        .map(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
```
결국 스트림 리스트 `List<Stream<String>>`이 만들어지면서 문제가 해결되지 않았습니다. 문제를 해결하려면  
먼저 각 단어를 개별 문자열로 이루어진 배열로 만든 다음에 각 배열을 별도의 스트림으로 만들어야 합니다.  
  
### flatMap 사용  
```java
List<String> flatMap = words.stream()
        .map(word -> word.split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(Collectors.toList());
```
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑합니다. 즉, map(Arrays::stream)과 달리 flatMap은 하나의 평면화된 스트림을 반환합니다.  
요약하자면, flatMap 메서드는 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행합니다.  
  
flatMap 메서드는 스트림의 형태가 배열과 같을 때, 모든 원소를 단일 원소 스트림으로 반환할 수 있습니다.  
스트림의 형태가 배열인 경우 또는 입력된 값을 또 다시 스트림의 형태로 반환하고자 할 때는 flatMap이 유용합니다.  
  
## 5.4 검색과 매핑
`특정 속성이 데이터 집합에 있는지 여부를 검색`하는 데이터 처리도 자주 사용됩니다.  
스트림 API는 **allMatch, anyMatch, noneMatch, findFirst, findAny**등 다양한 유틸리티 메서드를 제공합니다.  
  
### 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인
프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 anyMatch 메서드를 이용합니다.  
```java
 if (menu.stream().anyMatch(Dish::isVegetarian)) {
      System.out.println("The meu is (somewhat) vegetarian friendly!!");
    }
```
anyMatch는 불리언을 반환하므로 최종 연산입니다.  

### 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사  
allMatch 메서드는 anyMatch와는 달리 스트림의 모든 요소가 주어진 프레디케이트와 일치하는 지 검사합니다.  
```java
boolean isHealthy = menu.stream()
        .allMatch(dish -> dish.getCalories() < 1000);
```

### NONEMATCH
noneMatch는 allMatch와 반대 연산을 수행합니다. 즉, nonMatch는 주어진 프레디케이트와 일치하는 요소가 없는지 확인합니다.  
예를 들어 이전 예제를 다음처럼 noneMatch로 다시 구현할 수 있습니다.  
```java
boolean isHealthWithNoneMatch = menu.stream()
        .noneMatch(d -> d.getCalories() >= 1000);
```
anyMatch, allMatch, noneMatch 세 머서드는 스트림 쇼트서킷 기법(Java의 &&, ||)와 같은 연산을 활용합니다.  
마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산입니다.  
  
### 5.4.3 요소 검색 
findAny 메서드는 현재 스트림에서 **임의의 요소**를 반환합니다. findAny 메서드를 다른 스트림연산과 연결해서 사용할 수 있습니다.  
```java
Optional<Dish> dish = menu.stream()
        .filter(Dish::isVegetarian)
        .findAny();
```
스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화됩니다.  
즉, 쇼트서킷을 이용해서 결과를 찾는 즉시 실행을 종료합니다.  
  
### Optional이란?
`java.util.Optional`. `Optional<T>` 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스입니다.  
Optional은 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공합니다.  
+ isPresent()는 Optional이 값을 포함하면 참(true)을 반환하고, 값을 포함하지 않으면 거짓(false)을 반환합니다.  
+ `ifPresent(Consumer<T> block)`은 값이 있으면 주어진 블록을 실행합니다. Consumer : T -> void  
+ T get()은 값이 존재하면 값을 반환하고, 값이 없으면 NoSuchElementException을 일으킵니다.  
+ T orElse(T other)는 값이 있으면 값을 반환하고, 값이 없으면 기본값을 반환합니다.  

### 5.4.4 첫 번째 요소 찾기
```java
 List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
    Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
        .map(n -> n * n)
        .filter(n -> n % 3 == 0)
        .findFirst();

System.out.println(firstSquareDivisibleByThree.orElseThrow(NoSuchElementException::new));
```
병렬성 때문에 findFirst와 findAny 모두 필요하다. 병렬 실행에서는 첫 번째 요소를 찾기 어려우므로 반환 순서가 상관없다면  
병렬 스트림에서는 제약이 적은 findAny를 사용한다.  
  
## 리듀싱
스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법을 설명합니다.  
예를 들어, 메뉴의 모든 칼로리의 합계를 구하시오, 메뉴에서 칼로리가 가장 높은 요리는? 같은 질의.  
  
이러한 질의를 수행하려면 Integer 같은 결과가 나올 때까지 스트림의 모든 요소를 반복적으로 처리해야 합니다.  
이런 질의를 **리듀싱 연산(모든 스트림 요소를 처리해서 값으로 도출하는)**이라고 합니다.  

### 5.5.1 요소의 합
for-each 루프를 사용해서 리스트의 숫자 요소를 더하는 코드는 다음과 같습니다.  
```java
int sum = 0;
for (int x : numbers) {
    sum += x;
}
```
reduce를 이용하면 다음처럼 스트림의 모든 요소를 더할 수 있습니다.  
```java
  int sum = numbers.stream()
        .reduce(0, (a, b) -> a + b);
```
reduce로 다른 람다를 넘겨주면 모든 요소에 곱셈을 적용할 수 있습니다.  
```java
int product = numbers.stream()
        .reduce(1, (a, b) -> a * b);
```
메서드 참조를 이용하면 이 코드를 좀 더 간결하게 만들 수 있습니다.  
```java
int sum = numbers.stream()
        .reduce(0, Integer::sum);
```
  
### 초깃값 없음 
초깃값을 받지 않도록 오버로드된 reduce도 있다. 그러나 이 reduce는 Optional 객체를 반환합니다.  
```java
Optional<Integer> NoIdentity = numbers.stream().reduce((a, b) -> (a + b));
```
스트림에 아무 요소도 없는 상황이라면, 초기값이 없으므로 reduce는 합계를 반환할 수 없다. 따라서 합계가 없음을 가리킬 수 있도록  
Optional 객체로 감싼 결과를 반환합니다.  
  
### 5.5.2 최댓값과 최솟값
```java
//최댓값
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
    
//최솟값
Optional<Integer> min = numbers.stream()
    .reduce(Integer::min);
```
map과 reduce 메서드를 이용해서 스트림의 요리 개수를 계산할 수 있습니다.  
스트림의 각 요소를 1로 매핑한 다음에 reduce로 이들의 합계를 계산하는 방식으로 문제를 해결할 수 있습니다.  
```java
 int count = numbers.stream()
        .map(d -> 1)
        .reduce(0, (a, b) -> (a + b));
```
map과 reduce를 연결하는 기법을 맵 리듀스 패턴이라고 하며, (map - reduce pattern) 쉽게 병렬화하는 특징 덕분에 구글이 웹 검색에 적용하면서 유용해졌습니다.  
  
reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 됩니다.  
반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어렵습니다.  
강제적으로 동기화시키더라도 결국 병렬화로 얻어야 할 이득이 스레드 간의 소모적인 경쟁 때문에 상쇄되어 버린다는 사실을 알게 될 것입니다.  
  
사실 이 작업을 병렬화하려면 입력을 분할하고, 분할된 입력을 더한 다음에, 더한 값을 합쳐야 합니다.  
지금 중요한 사실은 가변 누적자 패턴은 병렬화와 거리가 너무 먼 기법이라는 점입니다.  
  
7장에서는 스트림의 모든 요소를 더하는 코드를 병렬로 만드는 방법도 설명합니다.  
`int sum = numbers.parallelStream().reduce(0, Integer::sum);`  
  
나중에 설명하겠지만 위 코드를 병렬로 실행하려면 대가를 지불해야 합니다.  
즉, reduce에 넘겨준 람다의 상태(인수턴수 변수 같은)가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야 합니다.  

### 스트림 연산 : 상태 없음과 상태 있음 
`map, filter` 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보냅니다.  
따라서(사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는다는 가정하에) 이들은 보통 상태가 없는,  
즉 내부 상태를 갖지 않는 연산(stateless operation)입니다.  
  
하지만 `reduce, sum, max`같은 연산은 결과를 누적할 내부 상태가 필요합니다.  
스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정(bounded)되어 있습니다.  
  
반면 `sorted, distinct` 같은 연산은 filter나 map처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것처럼 보일 수 있습니다.  
하지만 `sorted, distinct`는 filter나 map과는 다릅니다. 스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 합니다.  
  
예를 들어 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 합니다.  
연산을 수행하는 데 필요한 저장소 크기는 정해져있지 않습니다. 따라서 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있습니다.  
예를 들어 모든 소수를 포함하는 스트림을 역순으로 만들면?  
  
이러한 연산을 내부 상태를 갖는 연산이라고 합니다.  
  
## 5.6 실전 연습
예제에 사용한 Trader와 Transaction 클래스는 다음과 같습니다.  
```java
package com.me.modernJavainAction.chapter5;

import java.util.Objects;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@AllArgsConstructor
@Getter @Setter
public class Trader {
  private final String name;
  private final String city;

  @Override
  public int hashCode() {
    int hash = 17;
    hash = hash * 31 + (name == null ? 0 : name.hashCode());
    hash = hash * 31 + (city == null ? 0 : city.hashCode());
    return hash;
  }

  @Override
  public boolean equals(Object other) {
    if (other == this) {
      return true;
    }
    if (!(other instanceof Trader)) {
      return false;
    }
    Trader o = (Trader) other;
    boolean eq = Objects.equals(name,  o.getName());
    eq = eq && Objects.equals(city, o.getCity());
    return eq;
  }

  @Override
  public String toString() {
    return "Trader:" + this.name + " in " + this.city;
  }
}
```

```java
package com.me.modernJavainAction.chapter5;

import java.util.Objects;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@AllArgsConstructor
@Getter @Setter
public class Transaction {
  public final Trader trader;
  private final int year;
  private final int value;

  @Override
  public int hashCode() {
    int hash = 17;
    hash = hash * 31 + (trader == null ? 0 : trader.hashCode());
    hash = hash * 31 + year;
    hash = hash * 31 + value;
    return hash;
  }

  @Override
  public boolean equals(Object other) {
    if (other == this) {
      return true;
    }
    if (!(other instanceof Transaction)) {
      return false;
    }
    Transaction o = (Transaction) other;
    boolean eq = Objects.equals(trader,  o.getTrader());
    eq = eq && year == o.getYear();
    eq = eq && value == o.getValue();
    return eq;
  }

  @SuppressWarnings("boxing")
  @Override
  public String toString() {
    return String.format("{%s, year: %d, value: %d}", trader, year, value);
  }
}
```
다음과 같은 거래자(Trader) 리스트와 트랜잭션(Transaction) 리스트를 이용합니다.  
```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
        new Transaction(brian, 2011, 300),
        new Transaction(raoul, 2012, 1000),
        new Transaction(raoul, 2011, 400),
        new Transaction(mario, 2012, 710),
        new Transaction(mario, 2012, 700),
        new Transaction(alan, 2012, 950)
);
```
문제 풀이 :
```java
package com.me.modernJavainAction.chapter5;

import static java.util.Comparator.*;
import static java.util.stream.Collectors.*;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.stream.Collectors;

public class PuttingIntoPractice {

  public static void main(String[] args) {
    Trader raoul = new Trader("Raoul", "Cambridge");
    Trader mario = new Trader("Mario", "Milan");
    Trader alan = new Trader("Alan", "Cambridge");
    Trader brian = new Trader("Brian", "Cambridge");

    List<Transaction> transactions = Arrays.asList(
        new Transaction(brian, 2011, 300),
        new Transaction(raoul, 2012, 1000),
        new Transaction(raoul, 2011, 400),
        new Transaction(mario, 2012, 710),
        new Transaction(mario, 2012, 700),
        new Transaction(alan, 2012, 950)
    );

    //1번 : 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.
    List<Transaction> practice1 = transactions.stream()
        .filter(year -> year.getYear() == 2011)
        .sorted(comparing(Transaction::getValue))
        .collect(toList());
    System.out.println("practice 1 " + practice1);

    //2번 : 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.
    List<String> practice2 = transactions.stream()
        .map(Transaction::getTrader)
        .map(Trader::getCity)
        .distinct()
        .collect(toList());
    System.out.println("practice 2 " + practice2);

    //2번 추가
    transactions.stream()
        .map(transaction ->  transaction.getTrader().getCity())
        .distinct()
        .collect(toList());

    //3번 : 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.
    List<Trader> practice3 = transactions.stream()
        .map(Transaction::getTrader)
        .filter(trader -> trader.getCity().equals("Cambridge"))
        .distinct()
        .sorted(comparing(Trader::getName))
        .collect(toList());
    System.out.println("practice 3 " + practice3);

    //4번 : 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오
    String practice4 = transactions.stream()
        .map(transaction -> transaction.getTrader().getName())
        .distinct()
        .sorted()
        .reduce("", (n1, n2) -> n1 + n2 + " ");
    System.out.println("practice 4 " + practice4);

    //4번 joining 즉, StringBuilder 이용
    transactions.stream()
        .map(transaction -> transaction.getTrader().getName())
        .distinct()
        .sorted()
        .collect(joining());

    //5번 : 밀라노에 거래자가 있는가?
    boolean milanBased = transactions.stream()
        .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
    System.out.println("practice 5 " + milanBased);

    //6번 : 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오.
    System.out.print("practice 6 ");
    transactions.stream()
        .filter(t -> "Cambridge".equals(t.getTrader().getCity()))
        .map(Transaction::getValue)
        .forEach(i -> System.out.printf("%d ", i));
    System.out.println();

    //7번 : 전체 트랜잭션 중 최댓값은 얼마인가?
    Optional<Integer> practice7 = transactions.stream()
        .map(Transaction::getValue)
        .reduce(Integer::max);
    System.out.println("practice 7 " + practice7.orElseThrow(NoSuchElementException::new));

    //8번 : 전체 트랜잭션 중 최솟값은 얼마인가?
    Optional<Integer> practice8 = transactions.stream()
        .map(Transaction::getValue)
        .reduce(Integer::min);
    System.out.println("practice 8 " + practice8.orElseThrow(NoSuchElementException::new));

    //8번 : 람다 사용
    Optional<Transaction> practice88 = transactions.stream()
        .reduce((t1, t2) -> t1.getValue() < t2.getValue() ? t1 : t2);

  } //main END
}
```
스트림은 최댓값이나 최솟값을 계산하는 데 사용할 키를 지정하는 Comparator를 인수로 받는 min과 max 메서드를 제공합니다.  
따라서 min과 max를 이용하면 더 쉽게 문제를 해결할 수 있습니다.  
```java
Optional<Transaction> min = transactions.stream()
        .min(comparing(Transaction::getValue));
```
## 5.7 숫자형 스트림
reduce 메서드로 스트림 요소의 합을 구하는 예제를 살펴봤습니다.  
```java
int calories = menu.stream()
        .map(Dish::getCalories)
        .reduce(0, Integer::sum); //박싱 비용이 숨어있음 
```
사실 위 코드에는 박싱 비용이 숨어있습니다. 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야 합니다.  
다음 코드처럼 직접 sum 메서드를 호출할 수 있다면 더 좋지 않을까?  
```java
int calories = menu.stream()
                .map(Dish::getCalories)
                .sum();
```
하지만 위 코드처럼 sum 메서드를 직접 호출할 수 없습니다. sum 메서드가 `Stream<T>`를 생성하기 때문입니다.  
스트림의 요소 형식은 Integer지만 인터페이스에는 sum 메서드가 없습니다. 왜냐하면  
menu처럼 `Stream<Dish>` 형식의 요소만 있다면 sum이라는 연산을 수행할 수 없기 때문입니다.  
  
다행히도 스트림 API 숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림(Primitive Stream Specialization)을 제공합니다.  
### 5.7.1 기본형 특화 스트림
스트림 API는 박싱 비용을 피할 수 있도록 int요소에 특화된 `IntStream`, double 요소에 특화된 `DoubleStream`, long 요소에 특화된 `LongStream`  
을 제공합니다. 각각의 인터페이스는 숫자 스트림의 합계를 계산하는 sum, 최댓값 요소를 검색하는 max 같이 자주 사용하는  
숫자 관련 리듀싱 연산 수행 메서드를 제공합니다.  
  
또한 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공합니다.  
특화 스트림은 `오직 박싱 과정에서 일어나는 효율성`과 관련 있으며 스트림에 추가 기능을 제공하지는 않는다는 사실을 기억합니다.  
  
### 숫자 스트림으로 매핑 
스트림을 특화 스트림으로 변환할 때는 `mapToInt, mapToDouble, mapToLong` 세 가지 메서드를 가장 많이 사용합니다.  
이들 메서드는 map과 정확히 같은 기능을 수행하지만, `Stream<T>` 대신 특화된 스트림을 반환합니다.  
```java
int calories = menu.stream()
        .mapToInt(Dish::getCalories) //IntStream 반환 
        .sum();
```
mapToInt 메서드는 각 요리에서 모든 칼로리(Integer 형식)을 추출한 다음에 IntStream(`Stream<Integer>`)을 반환합니다.  
따라서 IntStream 인터페이스에서 제공하는 sum 메서드를 이용해서 칼로리 합계를 계산할 수 있습니다.  
  
스트림이 비어있으면 sum은 기본값 0을 반환합니다.  
  
### 객체 스트림으로 복원하기
숫자 스트림을 만든 다음에, 원상태인 특화되지 않은 스트림으로 복원할 수 있을까?  
IntStream은 기본형의 정수값만 만들 수 있습니다.  
IntStream의 map 연산은 'int를 인수로 받아서 int를 반환하는 람다(IntUnaryOperator)'를 인수로 받습니다.  
  
하지만 정수가 아닌 Dish 같은 다른 값을 반환하고 싶으면 어떻게 해야 할까요?  
`boxed` 메서드를 이용해서 특화 스트림을 일반 스트림으로 변환할 수 있습니다.  
```java
IntStream intStream = menu.stream()
        .mapToInt(Dish::getCalories);
    Stream<Integer> stream = intStream.boxed();
```
### 기본값 : OptionalInt
합계 예제에서는 0이라는 기본값이 있었으므로 별 문제가 없었습니다.  
하지만 IntStream에서 최댓값을 찾을 때는 0이라는 기본값 때문에 잘못된 결과가 도출될 수 있습니다.  
스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 어떻게 구별할 수 있을까요?  
  
값이 존재하는지 여부를 가리킬 수 있는 컨테이너 클래스 Optional 을 언급한 적이 있습니다.  
  
Optional을 Integer, String 등의 참조 형식으로 파라미터화할 수 있습니다.  
또한, `OptionalInt, OptionalDouble, OptionalLong` 세 가지 기본형 특화 스트림 버전도 제공합니다.  
  
다음처럼 OptionalInt를 이용해서 IntStream의 최댓값 요소를 찾을 수 있습니다.  
```java
OptionalInt maxCalories = menu.stream()
        .mapToInt(Dish::getCalories)
        .max();
```
이제 OptionalInt를 이용해서 최댓값이 없는 상황에 사용할 기본값을 명시적으로 정의할 수 있습니다.  
```java
//최댓값이 없을 때 1
int max = maxCalories.orElse(1);
```

### 5.7.2 숫자 범위
Java8의 IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공합니다.  
두 메서드 모두 첫 번째 인수로 시작값을, 두 번째 인수로 종료값을 가집니다.  
range : 시작값과 종료값이 결과에 포함되지 않습니다.  
rangeClosed : 시작값과 종료값이 결과에 포함됩니다.  
  
```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100) // [1,100]의 범위를 나타냅니다.
        .filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count()); //1부터 100까지에는 50개의 짝수가 있습니다.
```
filter를 호출해도 실제로는 아무 계산도 이루어지지 않습니다.  
최종적으로 결과 스트림에 count를 호출합니다.  
count는 최종 연산이므로 스트림을 처리해서 1부터 100까지의 숫자 범위에서 짝수 50개를 반환합니다.  
  
이때 rangeClosed 대신에 IntStream.range(1,100)을 사용하면 1과 100을 포함하지 않으므로 짝수 49개를 반환합니다.  
  
### 5.7.3 숫자 스트림 활용 : 피타고라스 수
피타고라스는 a * a + b * b = c * c 공식을 만족하는 세 개의 정수 (a, b, c)가 존재함을 발견했습니다.  
(c² = a² + b²)  

### 세 수 표현하기  
예를 들어 (3, 4, 5)를 new int[]{3, 4, 5}로 표현할 수 있습니다.  
  
### 좋은 필터링 조합
누군가 세 수 중에서 a, b 두 수만 제공했다고 가정해봅니다.  
두 수가 피타고라스 수의 일부가 될 수 있는 조합인지 어떻게 확인할 수 있을까요? a * a + b * b 의 제곱근이 정수인지 확인할 수 있습니다.  
```java
Math.sqrt(a*a + b*b) % 1 == 0;
```
이를 filter에 다음처럼 활용할 수 있습니다.  
```java
filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
```
위 코드에서 a라는 값이 주어지고 b는 스트림으로 제공된다고 가정할 때  
**filter로 a와 함께 피타고라스 수를 구성하는 모든 b를 필터링 할 수 있습니다.**  
  
### 집합 생성 
필터를 이용해서 옳은 조합을 갖는 a,b를 선택할 수 있게 되었습니다. 이제 마지막 세 번째 수를 찾아야 합니다.  
다음처럼 map을 이용해서 각 요소를 피타고라스 수로 변환할 수 있습니다.  
```java
stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
      .map(b -> new int[]{a, b, (int)Math.sqrt(a * a + b * b)}); //a와 b를 이용해서 나머지 수를 추출 
```
  
### b값 생성 
```java
 IntStream.rangeClosed(1, 100)
        .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
        .boxed()//객체 스트림으로 복원
        .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)});
```
filter 연산 다음에 rangeClosed가 반환한 IntStream을 boxed를 이용해서 `Stream<Integer>`로 복원했습니다.  
map은 스트림의 각 요소를 int 배열로 변환하기 때문입니다.  
IntStream의 map 메서드는 스트림의 각 요소로 int가 반환될 것을 기대하지만 이는 우리가 원하는 연산이 아닙니다.  
  
개체값 스트림을 반환하는 IntStream의 mapToObj 메서드를 이용해서 이 코드를 재구현할 수 있습니다.  
```java
IntStream.rangeClosed(1, 100)
         .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
         .mapToObj(b -> new int[]{a, b, (int)Math.sqrt(a*a + b*b)});
```
  
### a값 생성 및 최종 코드
```java
Stream<int[]> pythagoreanTriples = IntStream.rangeClosed(1, 100).boxed()
        .flatMap(a ->
            IntStream.rangeClosed(a, 100)
                .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
                .mapToObj(b ->
                    new int[]{a, b, (int) Math.sqrt(a * a + b * b)})
        );
```
flatMap은 어떤 연산을 수행하는 것일까요?  
우선 a에 사용할 1부터 100까지의 숫자를 만들었습니다.  
그리고 주어진 a를 이용해서 세 수의 스트림을 만듭니다.  
  
스트림 a의 값을 매핑하면 스트림의 스트림이 만들어질 것입니다.  
따라서 flatMap 메서드는 생성된 각각의 스트림을 하나의 평준화된 스트림으로 만들어줍니다.  
  
결과적으로 세 수로 이루어진 스트림을 얻을 수 있습니다.  
  
이제 limit을 이용해서 얼마나 많은 세 수를 포함하는 스트림을 만들 것인지만 결정하면 됩니다.  
```java
pythagoreanTriples.limit(5)
        .forEach(t -> System.out.println(t[0] + ", " + t[1] + ", " + t[2])); // T -> void
```

### 개선할 점?
현재 문제 해결 코드에서는 제곱근을 두 번 계산합니다.  
따라서 `(a*a, b*b, a*a+b*b)` 형식을 만족하는 세 수를 만든 다음에 우리가 원하는 조건에 맞는 결과만 필터링하는 것이 더 최적화된 방법입니다.  
```java
Stream<double[]> pythagoreanTriplesV2 = IntStream.rangeClosed(1, 100).boxed()
        .flatMap(a -> IntStream.rangeClosed(a, 100)
            .mapToObj(
                b -> new double[]{a, b, Math.sqrt(a * a + b * b)})
            .filter(t -> t[2] % 1 == 0)); // 세 수의 세 번째 요소는 반드시 정수여야 한다.
```

## 5.8 스트림 만들기 
stream 메서드로 컬렉션에서 스트림을 얻을 수 있었습니다. 그 뿐만 아니라 범위의 숫자에서 스트림을 만드는 방법도 설명했습니다.  
이 절에서는 일련의 값, 배열, 파일, 심지어 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림을 만드는 방법을 설명합니다.  
  
### 5.8.1 값으로 스트림 만들기
임의의 수를 인수로 받는 정적 메서드 Stream.of 를 이용해서 스트림을 만들 수 있습니다.  
예를 들어 다음 코드는 Stream.of로 문자열 스트림을 만드는 예제입니다.  
  
**스트림의 모든 문자열을 대문자로 변환한 후 문자열을 하나씩 출력합니다.**
```java
Stream<String> strings = Stream.of("Modern ", "Java ", "In ", "Action ");
strings.map(String::toUpperCase).forEach(System.out::println);
```
  
### 5.8.2 null이 될 수 있는 객체로 스트림 만들기
때로는 null이 될 수 있는 객체를 스트림(객체가 null이라면 빈 스트림)으로 만들어야 할 수 있습니다.  
예를 들어 System.getProperty는 제공된 키에 대응하는 속성이 없으면 null을 반환합니다.  
  
이런 메소드를 스트림에 활용하려면 다음처럼 null을 명시적으로 확인해야 했습니다.  
```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream
      = homeValue == null ? Stream.empty() : Stream.of(homeValue);
```
Stream.ofNullable을 이용해 다음처럼 코드를 구현할 수 있습니다.  
```java
Stream<String> homeValueStreamWithNullable 
        = Stream.ofNullable(System.getProperty("home"));
```
  
null이 될 수 있는 객체를 포함하는 스트림값을 flatMap과 함께 사용하는 상황에서는 이 패턴을 더 유용하게 사용할 수 있습니다.  
```java
Stream<String> values = Stream.of("config", "home", "user")
        .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
```
### 5.8.3 배열로 스트림 만들기
배열을 인수로 받는 정적 메서드 `Arrays.stream`을 이용해서 스트림을 만들 수 있습니다.  
예를 들어 다음처럼 기본형 int로 이루어진 배열을 IntStream으로 변환할 수 있습니다.  
```java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum(); //합계는 41
//IntStream으로 변환한 후 sum()을 돌려버림.
```

### 5.8.4 파일로 스트림 만들기 

















