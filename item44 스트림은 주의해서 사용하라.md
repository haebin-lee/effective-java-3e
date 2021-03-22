# item44. 스트림은 주의해서 사용하라.

스트림 API가 제공하는 핵심 
1. 데이터의 원소의 유한 혹은 무한 시퀀스를 의미
2. 스트림 파이프라인은 원소들로 수행하는 연산단계를 표현

스트림 파이프라인은 소스 스트림에서 시작해 **종단연산(terminal operation)**으로 끝나며, 그 사이에 하나 이상의 중간 **연산(intermediate operation)**이 있을 수 있다.

스트림 파이프 라인은 지연평가(lazy evaluation) 된다. **평가는 종단 연산이 호출될 때 이루어지며,** 종단연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 


사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
```java
public class Anagrams {

    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while(s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize) {
                System.out.println(group.size() + ": " + group);
            }
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- `computeIfAbsent`를 사용하면 각 키에 다수의 값을 매핑하는 맵을 쉽게 구현할 수 있다. 

스트림을 과하게 사용했다. - 따라하지 말것
```java
public class Anagrams {

    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted().collect(StringBuilder::new, (sb, c) -> sb.append((char)c), StringBuilder::append).toString())
            ).values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " +group)
                    .forEach(System.out::println);
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
- **스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다(이해하기도 쉽지 않다)**

스트림을 적절히 활용하면 깔끔하고 명료해진다.
```java
public class Anagrams {

    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .forEach(g -> System.out.println(g.size() +": " + g));
        }
    }
    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

**람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.**
**도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복 코드에서보다는 스트림 파이프라인에서 훨씬 크다.**

char 값들을 처리 할 때는 스트림을 삼가하는 편이 낫다

**기존 코드는 스트림을 사용하도록 리팩터링 하되, 새 코드가 더 나아 보일 때만 반영하자**

  
- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만을 읽을 수 있고, 지역변수를 수정하는 건 불가능하다. 
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다로는 이 중 어떤 것도 할 수 없다. 

