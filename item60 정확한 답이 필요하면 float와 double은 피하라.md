# item60. 정확한 답이 필요하다면 float와 double은 피하라

float와 double은 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계 되었다. 정확한 결과가 필요할 때는 사용하면 안되며 특히 금융 관련 계산과는 맞지 않다. 

오류 발생! 금융 계산에 부동 소수 타입을 사용했다.
```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemBought = 0;
    for (double price = 0.10 ; funds >= price ; price += 0.10) {
        funds -= price;
        itemBought++;
    }
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러):" + funds);
}
```
- 사탕 3개를 구입하고 남은 돈은 0.4여야 하지만 실제 결과는 0.3999999999999999이 나온다. 
- 금융계산에는 `BigDecimal`, `int` 혹은 `long`을 사용해야 한다.

BigDecimal을 사용한 해법. 속도가 느리고 쓰기 불편하다(값은 정확하다)
```java
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for (BigDecimal price = TEN_CENTS ; funds.compareTo(price) >= 0 ; price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought ++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(달러): " +funds);
}
```
- 기본 타입보다 쓰기가 훨씬 불편하고, 훨씬 느리다. 

BigDecimal의 대안으로 int 혹은 long 타입을 쓸 수도 있다.
```java
public static void main(String[] args) {
    int itemsBought = 0;
    int funds = 100;
    for (int price = 10 ; funds >= price ; price += 10) {
        funds -= price;
        itemsBought++;
    }
    System.out.println(itemsBought + "개 구입");
    System.out.println("잔돈(센트): " +funds);
}
```

소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라. 숫자를 아홉자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덟자리를 넘어가면 BigDecimal을 사용해라.