# 스트림 활용

## 필터링
#### 프레디케이트로 필터링

스트림 인터페이스는 *filter* 메서드를 지원한다, 이 메서드는 프레디케이트를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

```java
  List<Dish> vegetarianMenu = menu.stream().
                                  .filter(Dish::isVegetarian)   //채식 요리인지 확인하는 메서드 참조
                                  .collect(toList());
```

#### 고유 요소 필터링

*distcint* 메서드는 고유 요소로 이루어진 스트림을 반환한다.

```java
  List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
  numbers.stream()
         .filter(i -> i % 2 == 0)
         .distinct()                      //고유 요소 필터링
         .forEach(System.out::println);
```

## 스트림 슬라이싱
#### 프레디케이트를 이용한 슬라이싱

자바 9은 스트림의 요소를 효과적으로 선택할 수 있도록 *takeWhile, dropWhile* 두 가지 새로운 메서드를 지원한다.

#### takeWhile

다음과 같은 요리 목록을 갖고 있다고 가정하자.

```java
  List<Dish> specialMenu = Arrays.asList(
    new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
    new Dish("prawns", false, 300, Dish.Type.FISH),
    new Dish("rice", true, 350, Dish.Type.OTHER),
    new Dish("chicken", false, 400, Dish.Type.MEAT),
    new Dish("french fries", true, 530, Dish.Type.OTHER)
  );
```

이 리스트에서 320칼로리 이하의 요리를 선택할때 *filter* 연산을 사용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하게 된다. 따라서 리스트가 이미 정렬되어 있다면, 320칼로리보다 크거나 같은 요리가 나왔을 때 반복 작업을 중단할 수 있다. 이를 *takeWhile* 연산을 이용하면 간단하게 처리할 수 있다.

```java
  List<Dish> slicedMenu1
      = specialMenu.stream()
                   .takeWhile(dish -> dish.getCalories() < 320)
                   .collect(toList());
```

#### dropWhile

나머지 요소(320 칼로리보다 큰 요소)는 어떻게 선택할까?
바로 *dropWhile* 을 활용하면 된다.

```java
  List<Dish> slicedMenu2
      = specialMenu.stream()
                   .dropWhile(dish -> dish.getCalories < 320)
                   .collect(toList());
```

*dropWhile* 은 *takeWhile* 과 정반대의 작업을 수행한다. *dropWhile* 은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버리고 남은 요소들을 반환한다.

#### 스트림 축소

스트림은 주어진 값 이하의 크기(n)를 갖는 새로운 스트림을 반환하는 *limit(n)* 메서드를 지원한다. 간단하니까 코드로 살펴보자:

```java
  List<Dish> dishes = specialMenu.stream()
                                 .filter(dish -> dish.getCalories() > 300)
                                 .limit(3)
                                 .collect(toList());
```

정렬되지 않은 스트림에도 limit을 사용할 수 있지만 소스가 정렬 되어있지 않다면 limit의 결과도 정렬되지 않은 상태로 반환된다.

#### 요소 건너뛰기

스트림은 처음 n개 요소를 제외한 스트림을 반환하는 *skip(n)* 메서드도 지원한다. n개 이하의 요소를 포함하는 스트림에 skpi을 호출하면 빈 스트림이 반환된다.

```java
  List<Dish> dishes = menu.stream()
                          .filter(d -> d.getCalories() > 300)
                          .skip(2)
                          .collect(toList());
```

## 매핑
#### 스트림의 각 요소에 함수 적용하기

스트림은 함수를 인수로 받는 map 메서드를 지원한다. 다음은 Dish::getName을 map 메서드로 전달해서 스트림의 요리명을 추출하는 코드다.

```java
  List<String> dishNames = menu.stream()
                               .map(Dish::getName)
                               .collect(toList());
```

#### 스트림 평면화

리스트에서 고유 문자로 이루어진 리스트를 반환해보자. 예를 들어 ["Hello", "World"] 리스트가 있다면 결과로 ["H", "e", "l", "o", "W", "r", "d"]를 포함하는 리스트가 반환되어야 한다.

#### map과 Arrays.stream 활용

우선 문자열을 받아 스트림을 만드는 Arrays.stream() 메서드가 있다.
```java
  String[] arrayOfWords = {"Goodbye", "World"};
  Stream<String> streamOfWords = Arrays.stream(arrayOfWords);

  words.stream()
       .map(word -> word.split(""))       //각 단어를 개별 문자열 배열로 변환
       .map(Arrays::stream)               //각 배열을 별도의 스트림으로 생성
       .distinct()
       .collect(toList());
```
하지만 위 예제는 List<Stream<String>> 가 만들어지면서 문제가 해결되지 않았다.

#### flatMap 사용

flatMap을 사용하면 위 문제를 아래와 같이 해결할 수 있다.
```java
  List<String> uniqueCharacters = 
            words.stream()
                 .map(word -> word.split(""))     //각 단어를 개별 문자를 포함하는 배열로 변환
                 .flatMap(Arrays::stream)         //생성된 스트림을 하나의 스트림으로 평면화
                 .distinct()
                 .collect(toList());
```
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.

## 검색과 매칭

스트림 API는 allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메서드를 제공한다.

#### 프레디케이트가 적어도 한 요소와 일치하는지 확인

프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 anyMatch 메서드를 이용한다. 예를 들어 다음 코드는 menu에 채식요리가 있는지 확인한다.
```java
  if(menu.stream().anyMatch(Dish::isVegeterian)) {
    System.out.println("The menu is (somewhat) vegetarian friendly!!");
  }
```
anyMatch는 불리언을 반환하므로 최종 연산이다.

#### 프레디케이트가 모든 요소와 일치하는지 검사

allMatch 메서드는 anyMatch와 달리 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사한다.
```java
  boolean isHealthy = menu.stream()
                          .allMatch(dish -> dish.getCalories() < 1000);
```

#### noneMatch

noneMatch는 이름 그대로 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다. allMatch의 반대 연산이라고 생각하면 된다.
```java
  boolean isHealthy = menu.stream()
                          .noneMatch(d -> d.getCalories() >= 1000);
```

#### 요소 검색

findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다. 다음 코드처럼 filter와 findAny를 이용해서 채식 요리를 선택할 수 있다.
```java
  Optional<Dish> dish = menu.stream()
                            .filter(Dish::isVegetarian)
                            .findAny();
```

#### Optional이란?

Optional 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다. null은 쉽게 에러를 발생하므로, null이 반환될수 있는 상황에 Optional을 이용해서 null 확인 관련 버그를 피할 수 있다.

Optional이 제공하는 기능들:
  - isPresent()는 Optional이 값을 포함하면 참을 반환하고, 값을 포함하지 않으면 거짓을 반환한다.
  - ifPresent(Consumer<T> block)은 값이 있으면 주어진 블록을 실행한다.
  - T get()은 값이 존재하면 값을 반환하고, 값이 없으면 NoSuchElementException을 일으킨다.
  - T orElse(T other)는 값이 있으면 값을 반환하고, 값이 없으면 기본값을 반환한다.

위 기능을 활용한 코드를 살펴보자:
```java
  menu.stream()
      .filter(Dish::isVegetarian)
      .findAny()                //Optional<Dish> 반환
      .ifPresent(dish -> System.out.println(dish.getName()));     //값이 있으면 출력되고, 값이 없으면 아무일도 일어나지 않는다.
```

#### 첫 번째 요소 찾기

바로 코드로 살펴보자:
```java
  List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
  Optional<Integer> firstSquareDivisibleByThree =
                            someNumbers.stream()
                                       .map(n -> n * n)
                                       .filter(n -> n % 3 == 0)
                                       .findFirst();              //결과 값: 9
```

## 리듀싱
#### 요소의 합

reduce 메서드를 살펴보기 전에 for-each 루프를 이용해서 리스트의 숫자 요소를 더하는 코드를 확인하자.
```java
  int sum = 0;
  for (int x : numbers) {
    sum += x;
  }
```

우선 reduce는 두 인수를 받는다.
  - 초깃값
  - 스트림의 두 요소를 합쳐서 하나의 값으로 만드는 데 사용할 람다

reduce를 사용하면 아래와 같이 스트림의 모든 요소를 더할 수 있다.
```java
  int sum = numbers.stream().reduce(0, (a,b) -> a + b);
```

reduce로 다른 람다, 아래처럼 (a, b) -> a * b를 넘겨주면 모든 요소에 곱셈을 적용할 수 있다.
```java
  int product = numbers.stream().reduce(1, (a, b) -> a * b);
```
위 코드를 말로 설명하면 **스트림이 하나의 값으로 줄어들 때까지 각 요소를 반복해서 조합한다** 는 뜻이다.

초깃값을 받지 않도록 오버로드된 reduce도 있다. 그러나 이 reduce는 Optional 객체를 반환한다.
```java
  Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));
```
만약 스트림에 아무 요소도 없다면 초깃값이 없으므로 reduce는 합계를 반환할 수 없다. 따라서 합계가 없음을 가리킬수 있도록 Optional 객체로 감싼 결과를 반환하는 것이다.

**reduce를 이용하면 최댓값과 최솟값도 찾을 수 있다**
아래 코드는 스트림의 모든 요소를 소비할 때까지 람다를 반복 수행하면서 최댓값을 생산한다.
```java
  Optional<Integer> max = numbers.stream().reduce(Integer::max);
```
max대신 min을 넘겨주면 최솟값을 찾을 수 있다.

-----------------------------

## 실전 연습
트랙잭션을 실행하는 예제를 통해서 스트림 활용법을 알아보도록 하자.

아래는 예제들을 살펴보기 위한 클래스이다.
```java
public class Trader {

  private String name;
  private String city;

  public Trader(String n, String c) {
    name = n;
    city = c;
  }

  public String getName() {
    return name;
  }

  public String getCity() {
    return city;
  }

  public void setCity(String newCity) {
    city = newCity;
  }

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
    return String.format("Trader:%s in %s", name, city);
  }
}

public class Transaction {

  private Trader trader;
  private int year;
  private int value;

  public Transaction(Trader trader, int year, int value) {
    this.trader = trader;
    this.year = year;
    this.value = value;
  }

  public Trader getTrader() {
    return trader;
  }

  public int getYear() {
    return year;
  }

  public int getValue() {
    return value;
  }

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

아래 리스트 내용을 토대로 스트림을 활용해보자:
```java
public static void main(String... args) {
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
}
```

**1) 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하십시오.**
```java
    List<Transaction> practice1 = transactions.stream()
      .filter(d -> d.getYear() == 2011)
      .sorted(Comparator.comparing(Transaction::getYear))
      .collect(Collectors.toList());

    System.out.println("practice1 = " + practice1);
```
출력 값:
```
practice1 = [{Trader:Brian in Cambridge, year: 2011, value: 300}, {Trader:Raoul in Cambridge, year: 2011, value: 400}]
```

**2) 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.**
```java
    List<String> practice2 = transactions.stream()
      .map(d -> d.getTrader().getCity())
      .distinct()
      .collect(Collectors.toList());

    System.out.println("practice2 = " + practice2);
```
출력 값:
```
practice2 = [Cambridge, Milan]
```

**3) 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.**
```java
    List<Transaction> practice3 = transactions.stream()
      .filter(trans -> trans.getTrader().getCity().equals("Cambridge"))
      .sorted(Comparator.comparing(trans -> trans.getTrader().getName()))
      .collect(Collectors.toList());

    List<Trader> practice3_2 = transactions.stream()
      .map(Transaction::getTrader)
      .filter(a -> a.getCity().equals("Cambridge"))
      .sorted(Comparator.comparing(Trader::getName))
      .collect(Collectors.toList());
```
위 두 코드는 똑같이 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 나열한다. 하지만 practice3 은 결과 리스트가 Transaction 전체 정보를 담고 있고, practice3_2는 Transaction에서 Trader 정보만 추출해서 나열한다.

아래는 두 코드의 출력 값이다:
```
practice3 = [{Trader:Alan in Cambridge, year: 2012, value: 950}, {Trader:Brian in Cambridge, year: 2011, value: 300}, {Trader:Raoul in Cambridge, year: 2012, value: 1000}, {Trader:Raoul in Cambridge, year: 2011, value: 400}]

practice3_2 = [Trader:Alan in Cambridge, Trader:Brian in Cambridge, Trader:Raoul in Cambridge, Trader:Raoul in Cambridge]
```

**4) 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.**
```java
    List<String> practice4 = transactions.stream()
      .map(trans -> trans.getTrader().getName())
      .distinct()
      .sorted()
      .collect(Collectors.toList());

    System.out.println("practice4 = " + practice4);
```
출력 값:
```
practice4 = [Alan, Brian, Mario, Raoul]
```

**5) 밀라노에 거래자가 있는가?**
```java
    boolean practice5 = transactions.stream()
      .anyMatch(trans -> trans.getTrader().getCity().equals("Milan"));

    System.out.println("practice5 = " + practice5);
```
출력 값:
```
practice5 = true
```

**6) 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하십시오.**
```java
    transactions.stream()
      .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
      .map(Transaction::getValue)
      .forEach(System.out::println);
```
출력 값:
```
300
1000
400
950
```

**7) 전체 트랜잭션 중 최댓값은 얼마인가?**
```java
    Optional<Integer> practice7 = transactions.stream()
      .map(Transaction::getValue)
      .reduce(Integer::max);

    System.out.println("practice7 = " + practice7.get());
```
출력 값:
```
practice7 = 1000
```


## 숫자형 스트림
우리는 reduce 메서드로 스트림 요소의 합을 구하는 예제를 살펴봤다. 예를 들어 다음처럼 메뉴의 칼로리 합계를 계산할 수 있다:
```java
  int Calories = menu.stream()
    .map(Dish::getCalories)
    .reduce(0, Integer::sum);
```
하지만 위 코드는 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야하는 비용이 들어간다. 그래서 스트림 API는 숫자 스트림을 효율적으로 처리할 수 있도록 **기본형 특화 스트림 (primitive stream specialization)** 을 제공한다.

#### 기본형 특화 스트림
자바 8에서는 세 가지 기본형 특화 스트림을 제공한다: **IntStream, DoubleStream, LongStream**. 이 인터페이스들은 숫자 스트림의 합계를 계산하는 sum, 최댓값 요소를 검색하는 max 같이 유용한 숫자 관련 리듀싱 연산 메서드를 제공한다. 또한 이들을 필요할 때 다시 스트림으로 복원하는 기능도 제공한다.

**숫자 스트림으로 매핑**
스트림을 특화 스트림으로 변환할 때는 **mapToInt, mapToDouble, mapToLong** 세가지 메서드를 가장 많이 사용한다. map과 정확히 같은 기능을 수행하지만 Stream<T> 대신 특화된 스트림을 반환한다.

```java
  int calories = menu.stream()                        //Stream<Dish> 반환
                     .mapToInt(Dish::getCalories)     //IntStream 반환
                     .sum();
```
스트림이 비어있으면 sum은 기본값 0을 반환한다. sum외에도 max, min, average 등 다양한 유틸리티 메서드를 지원한다.

**객체 스트림으로 복원**
숫자 스트림을 원상태인 특화되지 않은 스트림으로 복원하려면 boxed 메서드를 이용해서 일반 스트림으로 변환할 수 있다.
```java
  IntStream intStream = menu.stream().mapToInt(Dish::getCalories);    //스트림 -> 숫자 스트림
  Stream<Integer> stream = intStream.boxed();                         //숫자 스트림 -> 스트림
```

**기본값 : OptionalInt**
스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 어떻게 구별할 수 있을까? 바로 **OptionalInt, OptionalDouble, OptionalLong** 등을 활용하면 된다. 물론 Optional을 Integer, String 등의 참조 형식으로 파라미터화 할수도 있다.

```java
  OptionalInt maxCalories = menu.stream()
                                .mapToInt(Dish::getCalories)
                                .max();
```
이제 최댓값이 없는 상황에 사용할 기본값을 명시적으로 정의할 수 있다.
```java
  int max = maxCalories.orElse(1);            //값이 없을 때 기본 최댓값을 명시적으로 설정
```

#### 숫자 범위
프로그램에서 특정 범위의 숫자를 이용해야 하는 상황이 자주 발생한다. 이럴때를 위해서 자바 8의 IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공한다. 두 메서드 모두 첫 번째 인수로 시작값을, 두 번째 인수로 종료값을 갖는다. 
```java
  IntStream evenNumbers = IntStream.rangeClosed(1, 100)       //[1, 100]의 범위를 나타낸다
                                   .filter(n -> n % 2 == 0);
  System.out.println(evenNumbers.count());                    //1부터 100까지에는 50개의 짝수가 있다.
```
위 코드처럼 rangeClosed를 이용해서 1부터 100까지의 숫자를 만들 수 있다. 이때 rangeClosed 대신에 IntStream.range(1, 100)을 사용하면 1과 100을 포함하지 않으므로 짝수 49개를 반환한다.

#### 숫자 스트림 활용 : 피타고라스 수
  **피타고라스 수** : (a * a) + (b * b) = (c * c) 공식을 만족하는 세 개의 정수.

**세 수 표현하기**
세 요소를 갖는 int 배열을 사용하면 쉽게 표현할 수 있다:
```java
  int[] pythagorean = new int[]{3,4,5};
```
이제 인덱스로 배열의 각 요소에 접근할 수 있다.

**좋은 필터링 조합**
누군가 세 수 중에서 a,b 두 수만 제공 했다고 가정하자. 두 수가 피타고라스 수의 일부가 될 수 있는지 확인 하려면 a * a + b * b의 제곱근이 정수인지 확인할 수 있다. 자바 코드로는 아래와 같다:
```java
  Math.sqrt(a*a + b*b) % 1 == 0;
```

만약에 a 라는 값이 주어지고 b는 스트림으로 제공이 된다고 가정할 때 아래 코드로 a와 함께 피타고라스 수를 구성하는 모든 b를 필터링할 수 있다.
```java
  filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
```

**집합 생성**
필터를 이용해서 좋은 조합을 갖는 a,b를 선택할 수 있게 되었다. 이제 마지막 세 번쨰 수를 찾아야 한다. 다음처럼 map을 이용해서 각 요소를 피타고라스 수로 변환할 수 있다.
```java
  stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
        .map(b -> new int[]{a, b, (int) Math.sqrt(a*a + b*b)});
```

**b값 생성**
이제 b값을 생성해야 한다. 코드로 살펴보자:
```java
  IntStream.rangeClosed(1, 100)
           .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
           .boxed()
           .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)});
```
filter 연산 다음에 rangeClosed가 반환한 IntStream을 boxed를 이용해서 Stream<Integer>로 복원했다. IntStream의 map 메서드는 스트림의 각 요소로 int가 반환될 것을 기대하지만 이는 우리가 원하는 연산이 아니다. 개체값 스트림을 반환하는 IntStream의 mapToObj 메서드를 이용해서 이 코드를 재구현할 수 있다:
```java
  IntStream.rangeClosed(1, 100)
           .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
           .mapToObj(b -> new int[]{a, b, (int) Math.sqrt(a*a + b*b)});
```

**a값 생성**
마지막으로 피타고라스 수를 생성하는 스트림을 완성하기 위해 a값을 생성하는 코드를 추가해보자:
```java
  Stream<int[]> pythagoreanTriples = IntStream.rangeClosed(1, 100).boxed()
        .flatMap(a ->
          IntStream.rangeClosed(a, 100)
            .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
            .mapToObj(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)})
        );
```
위 코드를 살펴보면 우선 1부터 100까지의 숫자를 만든후 주어진 a를 이용해서 세 수의 스트림을 만든다. 스트림 a의 값을 매핑하면 스트림의 스트림이 만들어지기 때문에 flatMap 메서드를 사용해서 생성된 각각의 스트림을 하나의 평준화된 스트림으로 만들었다. 그리고 b의 범위를 a ~ 100으로 설정해서 중복된 세 수의 생성을 방지했다.

**코드 실행**
이제 위 코드를 limit을 사용해서 얼마나 많은 세 수를 생성할지만 결정하면 된다.
```java
    pythagoreanTriples.limit(5)
      .forEach(t -> System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
```
출력 결과:
```
3, 4, 5
5, 12, 13
6, 8, 10
7, 24, 25
8, 15, 17
```

## 스트림 만들기
일련의 값, 배열, 파일 등 다양한 방식으로 스트림을 만드는 방법을 살펴보자.

#### 값으로 스트림 만들기
임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다. 다음은 Stream.of를 사용해서 문자열 스트림을 만드는 예제다.
```java
    Stream<String> stream = Stream.of("Java 8", "Lambdas", "In", "Action");
    stream.map(String::toUpperCase).forEach(System.out::println);
```

#### null이 될수 있는 객체로 스트림 만들기
자바 9에서 null이 될 수 있는 개체를 스트림으로 만들 수 있는 새로운 메서드가 추가되었다. 예를 들어 System.getProperty는 제공된 키에 대응하는 속성이 없으면 null을 반환한다. 이런 메서드를 스트림에 활용하려면 다음처럼 null을 명시적으로 확인해야 했다.
```java
    String homeValue = System.getProperty("home");
    Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);
```
Stream.ofNullable을 활용하면 다음처럼 구현할 수 있다:
```java
    Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

#### 배열로 스트림 만들기
배열을 인수로 받는 정적 메서드 Arrays.stream을 이용해서 스트림을 만들 수 있다. 다음은 기본형 int로 이루어진 배열을 IntStream으로 변환하는 코드다:
```java
    int[] numbers = { 2, 3, 5, 7, 11, 13 };
    System.out.println(Arrays.stream(numbers).sum());
```

#### 파일로 스트림 만들기
파일을 처리하는 등의 I/O 연산에 사용하는 자바의 NIO API도 스트림 API를 활용할 수 있다. 아래는 파일에서 고유한 단어 수를 찾는 코드다:
```java
    long uniqueWords = Files.lines(Paths.get("lambdasinaction/chap5/data.txt"), Charset.defaultCharset())
        .flatMap(line -> Arrays.stream(line.split(" ")))
        .distinct()
        .count();
```

#### 함수로 무한 스트림 만들기
스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드 Stream.iterate와 Stream.generate를 제공한다. 두 연산을 이용해서 **무한 스트림 (infinite stream) , 크기가 고정되지 않은 스트림,**을 만들 수 있다.

**iterate 메서드**
```java
  Stream.iterate(0, n -> n + 2)
        .limit(10)
        .forEach(System.out::println);
```
iterate 메서드는 초깃값과 람다를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다. 그렇기 때문에 주로 limit과 같이 사용이 된다. 예제의 스트림은 이전 결과에 2를 더한 값을 끊임없이 생상한다 (짝수). **무한 스트림**은 **언바운드 스트림 (unbound stream)** 이라고도 불린다.

아래는 **피보나치수열**을 생성하는 스트림이다.
```java
      Stream.iterate(new int[]{0, 1}, t -> new int[]{t[1], t[0] + t[1]})
            .limit(10)
            .map(t -> t[0])
            .forEach(System.out::println);
```

**generate 메서드**
generate도 요구할 때 값을 계산하는 무한 스트림을 만들 수 있다. 하지만 generate는 Supplier<T>를 인수로 받아서 새로운 값을 생산한다.

아래 코드는 0에서 1 사이의 임이의 더블 숫자 열 개를 만든다:
```java
  Stream.generate(Math::random)
        .limit(10)
        .forEach(System.out::println);
```

보다시피 generate 메서드도 limit을 사용해서 제한을 주지 않으면 **언바운드** 상태가 된다.