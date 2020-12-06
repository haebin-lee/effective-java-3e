# item42. 익명 클래스보다는 람다를 사용하라

추상메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용 
-> 이런 인터페이스의 인스턴스를 함수객체(function object) 라고 하여 특정 함수나 동작을 나타내는데 사용했다. 

예를들어 문자열의 길이 순으로 정렬할 때 아래처럼 사용했다(Me too)
```java
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length())
    }
})
```

java8 에서 추상 메서드 하나짜리는 람다식으로 만들어서 사용할 수 있게 되었다. 

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.lenght(), s2.length()))
```
- 컴파일러가 문맥을 살펴서 타입을 추론해준다 
- 상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다 
- **타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입을 생략하자**


위 코드를 더 간결하게 만들 수 있다. 
```java
Collections.sort(words, comparingInt(String::length))
```

List 인터페이스에 추가된 sort 메소드를 이용하면 더 짧아진다 
```java
words.sort(comparingInt(String::length))
```

