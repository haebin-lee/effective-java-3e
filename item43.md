# item43. 람다보다는 메서드 참조를 사용하라.

간결함

익명클래스 < 람다 < 메서드 참조 

람다는 길이는 더 길지만 메서드 참조보다 읽기 쉽고 유지보수도 쉬울 수 있다. 

람다
```java
map.merge(key, 1, (count, incr)-> count+incr);
```
메서드 참조 
```java
map.merge(key, 1, Integer::sum);
```

항상 그런건 아니다 

람다
```java
service.execute(GoshThisClassNameIsHumongous::action);
```

메서드 참조
```java
service.execute(() -> action())
```

메서드 참조의 유형 
| 메스더 참조 유형 | 예 | 같은 기능을 하는 람다 | 
|---|:---:|---:|
| 정적 | Integer::parseInt | str -> Integer.parseInt(str)|
| 한정적(인스턴스) | Instant.now()::isAfter | Instant then = Instant.now(); t-> then.isAfter |
| 비한정적(인스턴스) | String::toLowerCase | str-> str.toLowerCase() |
| 클래스 생성자 | TreeMap<K, V>::new | () -> new TreeMap<K, V>() |
| 배열 생성자 | int[]::new | len -> new Int[len] |

```
핵심정리 
메서드 참조는 람다의 강단 명료한 대안이 될 수 있다. 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고 그렇지 않을 때만 람다를 사용하라
```