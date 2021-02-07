# item58. 전통적인 for문 보다는 for-each 문을 사용하라

컬렉션 순회하기 - 더 나은 방법이 있다. 
```java
for (Iterator<Element> i = c.iterator() ; i.hasNext() ; ) {
    Element e = i.next();
    //  e로 무언가를 한다.
}
```
배열 순회하기 - 더 나은 방법이 있다.
```java
for(int i = 0 ; i < a.length ; i ++) {
    // a[i] 로 무언가를 한다.
}
```

위 관용구들은 while문 보다는 낫지만 가장 좋은 방법이 아니다. > for-each문으로 모두 해결할 수 있다.
- 인덱스 변수를 사용하지 않으므로 코드가 깔끔해지고 오류가 날 일도 없다. 
- 하나의 관용구로 컬렉션과 배열을 모두 처리 할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다. 

컬렉션과 배열을 순회하는 올바른 관용구
```java
for (Element e : elements) {
    // e로 무언가를 한다.
}
```
반복대상이 컬렉션이든 배열이든, for-each문을 사용해도 속도는 그대로다. 

버그를 찾아보자
```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum RANK { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>(); 
for (Iterator<Suit> i = suits.iterator() ; i.hasNext() ; ) 
    for (Iterator<Rank> j = ranks.iterator() ; j.hasNext() ; ) 
        deck.add(new Card(i.next(), j.next()));
```
`next()`는 숫자(Suit) 당 하나씩 불려야 하는데 안쪽 반복문에 의해 카드(Rank) 하나당 한번씩 불리고 있어서 숫자가 바닥나면 NoSuchElementException을 던진다 

문제는 고쳤지만 보기 좋진 않다 더 나은 방법이 있다! 
```java
for (Iterator<Suit> i = suits.iterator() ; i.hasNext() ; ) {
    Suit suit = i.next(); 
    for (Iterator<Rank> j = ranks.iterator() ; j.hasNext(); ) {
        deck.add(new Card(suit, j.next()));
    }
}
```

컬렉션이나 배열의 중첩 반복을 위한 권장 관용구
```java
for (Suit suit : suits) {
    for (Rank rank : ranks) {
        deck.add(new Card(suit, rank));
    }
}
```

for-each를 사용할 수 없는 경우는 다음과 같다. 
1. 파괴적인 필터링(destructive filtering) : 컬렉션을 순회하면서 원소를 제거해야한다면 반복자의 remove메서드를 호출해야 한다. 자바8 부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다. 
2. 변형(transforming) : 일부를 교체하거나 전체를 교체해야 한다면 인덱스를 사용해야 한다. 
3. 병렬 반복(parallel iterator) : 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어 해야 한다. 

핵심 정리 
전통적인 for문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없다. 가능한 모든 곳에서 for문이 아닌 for-each문을 사용하자.