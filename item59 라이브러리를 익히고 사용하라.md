# item59. 라이브러리를 익히고 사용하라

흔하지만 문제가 심각한 코드!
```java
static Random  rnd = new Random(); 

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```
문제점 
- n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다. 
- n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다. 
- n 값이 크면 이 현상은 더 두드러진다. 

```java
public static void main(String[] args) {
    int n = 2 * (Integer.MAX_VALUE / 3);
    int low = 0; 
    for (int i = 0 ; i < 1000000 ; i++) {
        if (random(n) < n / 2){
            low ++; 
        }
    }
    System.out.println(low);
}
```
- 무작위로 생성된 수 중에서 2/3가량이 중간값보다 낮은 쪽으로 쏠린다. 

위 문제는 `Random.nextInt(int)` 함수를 사용하면 모두 해결이 된다. 

**표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.**
자바7 부터는 Random을 더이상 사용하지 않는게 좋다. `ThreadLocalRandom`으로 대체하면 잘 작동한다. 심지어 속도도 더 빠르다. 한편, 포크-조인풀이나 병렬 스트림에서는 `SplittableRandom`을 사용하라

메이저 릴리즈마다 주목할 만한 수많은 기능이 라이브러리에 추가된다.


transferTo 메서드를 이용하여 URL의 내용 가져오기 - 자바9부터 가능하다.
```java
public static void main(String[] args) throws IOException {
    try (InputStream in = new URL(args[0]).openStream()) {
        in.transferTo(System.out);
    }
}
```

자바 프로그래머라면 적어도 `java.lang`, `java.util`, `java.io`와 그 하위 패키지들에는 익숙해 져야 한다. 
`java.util.concurrent`의 동시성 기능도 마찬가지다. 

