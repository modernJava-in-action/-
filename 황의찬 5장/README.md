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
