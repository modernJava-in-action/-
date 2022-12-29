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











