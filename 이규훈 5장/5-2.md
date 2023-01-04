# 복습
>스트림이란?
람다를 활용할 수 있느 기술 중 하나이다.
왜 사용? 
--
배열이나 컬렉션에 접근할때 반복문이나 iterator를 사용하여 매번 새로운 코드를 작성해야한다.
이는 매우 귀찮고 길이도 길어지고 가독성도 떨어진다. 그리고 결정적을 코드의 재사용이 불가능하다.


> 컬렉션이란?
컬렉션 프레임워크(collection framework)란?
자바에서 컬렉션 프레임워크(collection framework)란 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미합니다
즉, 데이터를 저장하는 자료 구조와 데이터를 처리하는 알고리즘을 구조화하여 클래스로 구현해 놓은 것입니다.
이러한 컬렉션 프레임워크는 자바의 인터페이스(interface)를 사용하여 구현됩니다. 인터페이스(interface)란 다른 클래스를 작성할 때 기본이 되는 틀을 제공하면서, 다른 클래스 사이의 중간 매개 역할까지 담당하는 일종의 추상 클래스를 의미합니다. 
출처: TCPSCHOOL
인터페이스는 사실상 다중상속을 위해 쓰는 것

## 스트림 API의 특징
1. 스트림은 외부 반복을 통해 작업하는 컬렉션과는 달리 내부 반복(internal iteration)을 통해 작업을 수행합니다.
- 외부 반복: 사용자가 직접 반복문 돌리는거
- 내부 반복: 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는 것
```
// 외부 반복
List<String> list = new ArrayList<>();
Iterator<String> iter = member.iterator();

while(iter.hasNext()){
    Member member = iter.next();
    if(member.getAge() >= 20){
        list.add(member.getName());
    }
}

// 내부 반복
List<String> list = member.stream()
    				.filter(m -> m.getAge() >= 20)
    				.collect(toList());
```

2. 스트림은 재사용이 가능한 컬렉션과는 달리 단 한 번만 사용할 수 있습니다.

3. 스트림은 원본 데이터를 변경하지 않습니다.

4. 스트림의 연산은 필터-맵(filter-map) 기반의 API를 사용하여 지연(lazy) 연산을 통해 성능을 최적화합니다.
> 지연(lazy)연산이란?
지연 연산이란 간단히 말해 결과값이 필요할 때까지 계산을 늦추는 기법이다. 즉, 눈앞에 코드가 주어졌을 때 곧바로 해당 코드를 실행하는 것이 아니라 실행 결과가 필요해지는 시점에 실행하도록 하는 것이다.
5. 스트림은 parallelStream() 메소드를 통한 손쉬운 병렬 처리를 지원합니다. 

결국 스트림은 계산에 용이한 API이다.

# 5.7 숫자형 스트림
칼로리의 합계를 구하는 코드
```
int cal = menu.stream()
			  .map(Dish::getCalories)
              .reduce(0, Integer::sum);
```
여기서 박싱비용이 있다.
>박싱이란?
갑 형식을 참조 형식으로 변환하는 것
ex) int -> Object
언박싱 = 참조 형식을 값 형식으로 변환하는 것

>참조형이란?
긱본형 제외한 모든 타입
기본형-> int,long,char,double,boolean 등등

여기서는 합계를 계산하기 위해
Integer에서 기본형으로 바꿀때 발생한다.
바로 .sum()를 호출하면 되지만 이는 불가능하다.
왜냐하면 인터페이스에 sum이 없기때문이다.

이러한 박싱 비용을 피할 수 있게 기본형 특화 스트림을 제공한다.

## 기본형 특화 스트림

1) int 요소에 특화된 IntStream
2) long 요소에 특화된 LongStream
3) Double 요소에 특화된 DoubleStream

특화 스트림은 오직 박싱과정에서 일어나는 효율성과 관련있으며 스트림에 추가기능을 제공하지는 않는다.

### 숫자 스트림으로 매핑
`maptoInt`, `mapToDouble`, `mapToLong` 세 가지 메서드를 가장 많이 사용한다. 이들은 map과 똑같은 기능을 제공하지만 `Stream<T>` 대신 특화된 스트림을 반환한다.
이를 이용해서 앞의 코드를 다시 짜보면

```
int cal = menu.stream()
			  .mapToInt(Dish::getCalories)
              .reduce(0, Integer::sum);
```
mapToInt는 각 요리에서 모든 칼로리 추출하고 IntStream을 반환한다.
이러면 sum메서드를 사용가능해진다. 

### 객체 스트림으로 복원하기

숫자 스트림을 만든 이후에, 원상태인 특화되지 않은 스트림으로 복원 할 수 있을까? 이럴 경우에는 boxed메서드를 이용해서 특화 스트림을 일반 스트림으로 변환 할 수 있다.
```
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
// 스트림을 숫자스트림으로 변환한다.
Stream<Integer> stream = inStream.boxed();// 숫자 스트림을 스트림으로 변환한다.
```

### 기본값: Optionallnt
> 숫자 스트림에서 최댓값을 찾을때 이게 기본값 0인지 진짜 찾고보니 0이었는지 어떻게 구분해야할까?

이때에는 값이 존재하는지 여부를 알려주는 컨테이너 클래스 `Optional`을 사용하면 된다.
Optional을 Integer, String 등의 참조 형식으로 파라미터화 할 수 있다. 또한 OptionalDouble, OptionalLong 세가지 기본형 특화 스트림버전도 있다.

```
OPtional maxCalories = menu.stream()
						   .mapToInt(Dish::getCalories)
                           .max();
                           
int max = maxCalories.orElse(1); // 값이 없을때 기본 최댓값을 명시적으로 설정
````

## 숫자 범위
range,rangeClosed라는 정적 메서드 제공한다.
첫번째 인수로는 시작값
두 번째 인수로는 종료값\
```
IntStream.rangeClosed(1,100) // [1,100] 범위 나타낸다
```

## 피타고라스 수 : 숫자 스트림 활용
![](https://velog.velcdn.com/images/normalvector/post/02349645-1d6b-4b22-b3ac-798d121db48c/image.png)

### 세 수 표현하기
세 수를 정의하는 것부터 시작하면
int 배열로 정의한다.
```
new int[]{3,4,5}
```
### 좋은 필터링 조합
만약 세가지 수 중에서 a,b 두 수 만 제공했다고 가정하자. 두 수가 피타고라스 수의 일부가 될 수 있는 좋은 조합인지 어떻게 확인할 수 있는가?
이는 a * a+ b * b의 제곱근이 정수인지 확인할 수 있다. 자바 코드로는 다음과 같이 구현할 수 있다.
```
Math.sqrt(a*a + b *b ) % 1==0;
```
이때 x가 부동 소숫점 이라면 x%1.0이라는 자바 코드로 소숫점 이하 부분을 얻을 수 있다. 예를 들어 5.0이라는 수에 이 코드를 적용하면 소숫점 이하는 0이 된다.
이를 filter에 다음처럼 활용할 수 있다.
```
filter( b-> Math.sqrt(a*a + b *b) % 1==0)
```
### 집합 생성
필터를 이용해서 좋은 조합을 갖는 a,b를 선택할 수 있다.
```
stream.filter( b-> Math.sqrt(a*a + b *b) % 1==0)
	  .map(b -> new int[]{a,b, (int)Math.sqrt(a * a + b *b)});
```
### b값 생성
```
IntStream.rangeClosed(1, 100)
         .filter( b-> Math.sqrt(a*a + b *b) % 1==0)
         .boxed()
	     .map(b -> new int[]{a,b, (int)Math.sqrt(a * a + b *b)});	
```
boxed를 이용해서 `Stream<Integer>`로 복원한다.
map은 스트림의 각 요소를 int 배열로 변환하기 때문이다. 
IntStream의 map메서드는 스트림의 각 요소로 int가 반환될 것을 기대하지만 이는 우리가 원하는 연산의 결과가 아니다
이를 위해서 우리는 IntStream을 Stream로 변환할 때 사용하는 `mapToObj()` 사용한다.

```
IntStream.rangeClosed(1, 100)
         .flatMap(b -> IntStream.rangeClosed(a, 100)
         .mapToObj(b -> new double[]{a, b, Math.sqrt(a * a + b * b)})
```

### a값 생성
a는 b와 비슷한 방법으로 생성할 수 있다.
```
Stream<int[]> pythagoreanTriples = IntStream.rangeClosed(1, 100).boxed()
        .flatMap(a -> 
        	IntStream.rangeClosed(a, 100)
                     .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
                   .mapToObj(b -> 
                   				new int[]{a, b, Math.sqrt(a * a + b * b)}));
```
여기서 flatMap은 생성된 각각의 스트림을 하나의 평준화된 스트림으로 만들어준다.
이러면 결국 세 수로 이루어진 스트림을 얻을 수 있다.


# 스트림 만들기


## 값으로 스트림 만들기
Stream.of를 이용해서 스트림을 만들 수 있다.
Stream.of는 임의의 수를 인수로 받는 정적 메서드이다.
empty 메서드를 이용해서 스트림을 비울 수 있다.

## null이 될 수 있는 객체로 스트림 만들기
null이 될 수 있는 개체를 스트림을 만들 수 있는 새로운 메소드가 필요하다.
Stream.ofNullable을 이용해서 대응하는 것이 없으면 null을 반환할수 있다.

## 배열로 스트림 만들기
배열을 인수로 받는 Arrays.stream을 이용해서 스트림을 만들 수 있다.


## 함수로 무한 스트림 만들기
Stream.iterate
Stream.generate
이 두 연산을 이용해서 무한 스트림, 즉 크기가 고정되지 않은 스트림을 만들 수 있다.
언바운드 스트림이라고도 한다.
### iterate 메서드
```
Stream.iterate(0, n->n+2)
      .limit(1)
      .forEach(System.out::println);
```
iterate메서드는 초깃값과 람다를 인수로 받아서 새로운 값을 끊임없이 n -> n + 2 즉 이전 결과에 2를 더한 값 이므로 짝수를 반환한다.



### generate 메서드
generate도 무한 스트림을 만들 수 있다. 그러나 **생산된 값을 연속적으로 계산하지않는 다는 점**에서 iterator와 다르다. 
Stream.generate()는 인자로 함수를 받고, 이 함수가 리턴하는 객체들을 요소로 갖는 Stream을 생성한다. 이 함수는 인자를 받지 않고 리턴 값이 있는 함수입니다.
Stream.iterate()도 generate()와 유사합니다. 하지만 두개의 인자를 받습니다. 첫번째 인자는 초기값, 두번째 인자는 함수입니다. 이 함수는 1개의 인자를 받고 리턴 값이 있습니다.

