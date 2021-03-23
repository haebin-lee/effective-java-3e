# item46 스트림에서는 부작용 없는 함수를 사용하라.

각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 **순수함수**여야 한다.
스트림의 연산 내에서는 함수 객체는 모두 부작용(side effect)가 없어야 한다.


스트림의 패러다임을 이해하지 못한채 API만 사용했다. - 따라하지 말 것 
```java
Map<String, Long> freq = new HashMap<>(); 
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```
위 코드는 스트림 코드를 가장한 반복 코드이다. 특히 forEach내부에서 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다. 


스트림을 제대로 활용해 빈도표를 초기화 한다.
```java
Map<String, Long> freq; 

try (Scanner<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다. 대놓고 반복적이라서 병렬화할 수도 없다. **forEach 연산은 스트림 계산 결과 단계를 보고할 때만 사용하고, 계산하는데는 쓰지 말자.** 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로 쓸 수 있다.

빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
```java
List<String> topTen = freq.keySet().stream()
        .sorted(Comparator.comparing(freq::get).reversed())
        .limit(10)
        .collect(Collectors.toList());
```

toMap 수집기를 사용하여 문자열을 열거타입 상수에 매핑한다.
```java
 private static final Map<String, Operation> stringtoEnum =
    Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기 
```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a->a, maxBy(comparing(Album::sales)));
)
```
-> 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다.


마지막에 쓴 값을 취하는 수집기
```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```
`groupingBy`는 입력으로는 분류 함수(classifer)를 받고 출력으로는 원소들을 카테고리로 모아 놓은 맵을 담은 수집기를 반환한다.

`partitioningBy` 분류함수 자리에 프레디키트(predicate)를 받고 키가 boolean인 맵을 반환한다. 

`joining`는 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다. charsequence에서만 적용 가능하다. 


핵심정리 

스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 스트림 뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다. 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다. 가장 중요한 수집기 팩터리는 `toList`, `toSet`, `toMap`, `groupingBy`, `joining` 이다.



