# 이왕이면 제네릭 메서드로 만들라 (Item 30)
  
- - - -

### 제네릭 메서드로 구현된 유틸리티 메서드

유틸리티 메서드는 보통 제네릭 메서드인 경우가 많다.  

```
public static Set union (Set s1, Set s2){
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

- 컴파일은 가능하나 컴파일 타임에 raw 타입의 Set의 구성에 대한 검사가 불가능해 경고가 발생한다

```
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

- 제네릭을 사용하면 타입에 안전하게 컴파일 도중 검사가 가능하다.
  
  
- - - -
### 제네릭 싱글턴 팩토리

```
static <E> Set<E> of(E e1, E e2) {
    return new ImmutableCollections.Set12<>(e1, e2);
}
```

- 많이 사용하는 팩터리 또한 요청 타입에 맞게 변환되는 제네릭을 사용한다.
  
  
```
static final <T> Set<T> emptySet() {
    return (Set<T>) EMPTY_SET;
}
```
  
- 인스턴스를 만들어 둔 뒤에 사용 시에 타입을 결정해 반환이 가능해진다.
  

- - - - 
  
### 재귀적 타입 한정을 사용

```
public static <E extends Comparable<E>> E max(Collection<E> c);
```
  
조금 특별한 사용법으로 자기 자신이 포함된 표현식(`Comparable<E>`)을 사용해 입력되는 타입을 한정할 수 있다.
위의 메서드는 `E`가 사용된 컬렉션을 입력으로 받으나 해당 타입 `E`를 상호 비교가 가능한 것으로 한정하고 있다.<br>
  
  
  
✔ API 설계에선 사용자가 형변환을 하지 않을 수 있도록 안전하게 설계하자.


### 당장은 제네릭 클래스, 메서드를 사용할 일이 없을까?
  
  
[템플릿 콜백 패턴](https://jaehee329.tistory.com/23)을 사용한 예외 처리