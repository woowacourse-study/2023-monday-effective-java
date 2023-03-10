# [Item50] 적시에 방어적 복사본을 만들라

자바는 안전한 언어 but 악의적인 의도 혹은 실수로 오작동이 가능
-> 방어적 복사를 통한 방어

## 용어
### 방어적 복사 != 깊은 복사

- 방어적 복사 : 원본 리스트와의 연관 관계를 끊는 것
- 깊은 복사 : 원본 리스트와의 관계 뿐만 아니라 내부 요소의 관계들도 다 끊는 것

### 불변식
- 객체가 정상적으로 작동하기 위해서 항상 참이 되어야 하는 조건
  - ex) Deck -> 카드가 항상 1장 이상 있어야 정상 작동

## `new ArrayList<>()` vs `List.copyOf()` vs `Collection.unmodifiableList()`

### [예시 코드]

- `Name`
    - String name
    - getter, setter, toString 정의
- `Names`
    - List\<Name> names
    - void add(final Name name)
    - getter, toString 정의

```java
@Test
void 주소_값을_비교한다(){
    // given
    final Name 에단 = new Name("에단");
    final Name 홍실 = new Name("홍실");

    final List<Name> beforeNames=new ArrayList<>();
    beforeNames.add(에단);
    beforeNames.add(홍실);

    // when, then
    // 원래 주소
    System.out.println("===========before===========");
    System.out.println("before List = " + System.identityHashCode(beforeNames));
    System.out.println("에단 = " + System.identityHashCode(에단));
    System.out.println("홍실 = " + System.identityHashCode(홍실));
    System.out.println();
    
    final Names names = new Names(new ArrayList<>(beforeNames)); // 추가 가능, 원본과 연결 끊김
    //        final Names names = new Names(List.copyOf(beforeNames)); // 추가 불가, 원본과 연결 끊김
    //        final Names names = new Names(Collections.unmodifiableList(beforeNames)); // 추가 불가, 원본과 연결 됨
    final List<Name> afterNames = names.getNames();

    // 나중 주소
    System.out.println("===========after===========");
    System.out.println("after List = " + System.identityHashCode(afterNames));
    System.out.println("에단 = " + System.identityHashCode(afterNames.get(0)));
    System.out.println("홍실 = " + System.identityHashCode(afterNames.get(1)));
    System.out.println();

    beforeNames.add(new Name("에밀"));
    //        names.add(new Name("오리"));
    홍실.setName("모디");

    final Name ethan = names.getNames().get(0);
    ethan.setName("오리");

    System.out.println("===========list 비교========");
    System.out.println("원래 리스트 : " + beforeNames);
    System.out.println("Names : " + names.getNames());
}
```

### [실행 결과]

- `final Names names = new Names(new ArrayList<>(beforeNames));` ->  추가 가능, 원본과 연결 끊김
- `names.add(new Name("오리"));` -> 정상 동작, 원본과 연결이 끊겼으므로 Names만 추가
```java
===========before===========
before List = 1346201722
에단 = 2011997442
홍실 = 843512726

===========after===========
after List = 1631086936
에단 = 2011997442
홍실 = 843512726

===========list 비교========
원래 리스트 : [Name{name='디투'}, Name{name='모디'}, Name{name='에밀'}]
Names : [Name{name='디투'}, Name{name='모디'}, Name{name='오리'}]
```

<br/>

- `final Names names = new Names(List.copyOf(beforeNames));` ->  추가 불가, 원본과 연결 끊김
- `names.add(new Name("오리"));` -> 예외 발생
```java
===========before===========
before List = 1346201722
에단 = 2011997442
홍실 = 843512726

===========after===========
after List = 1631086936
에단 = 2011997442
홍실 = 843512726

===========list 비교========
원래 리스트 : [Name{name='디투'}, Name{name='모디'}, Name{name='에밀'}]
Names : [Name{name='디투'}, Name{name='모디'}]
```

<br/>

- `final Names names = new Names(Collections.unmodifiableList(beforeNames));` ->  추가 불가, 원본과 연결 됨 
- `names.add(new Name("오리"));` -> 예외 발생
```java
===========before===========
before List = 1346201722
에단 = 2011997442
홍실 = 843512726

===========after===========
after List = 1631086936
에단 = 2011997442
홍실 = 843512726

===========list 비교========
원래 리스트 : [Name{name='디투'}, Name{name='모디'}, Name{name='에밀'}]
Names : [Name{name='디투'}, Name{name='모디'}, Name{name='에밀'}]
```

### 결론

|     구분     | new ArrayList<>() | List.copyOf() | Collection.unmodifiableList() |
|:----------:|:-----------------:|:-------------:|:-----------------------------:|
| 복사 후 추가 여부 |         O         |       X       |               X               | 
|  원본과의 연결   |         X         |       X       |               O               |

- 리스트안의 원소가 100000개일 때 시간 비교(nano)
  - new ArrayList : 1134125
  - List copyOf : 4022667
  - unmodifiableList : 1353708
  - ArrayList + unmodifiableList : 1011000

## 방어적 복사 활용 예

### 생성자에서

```java
public Cards(final List<Card> cards) {
    this.cards = new ArrayList<>(cards);
}
```

### getter에서

```java
public List<Card> getCards() {
    return List.copyOf(cards);
}
```

## 멀티 스레딩에서 방어적 복사

- TOCTOU 공격 방지(검사시점/사용시점 공격) 방지

[AS-IS]

```java
public Name(final String name) {
    validate(name)      // 이 시점에서는 정상
                        // 요 사이에서 다른 쓰레드가 변경 가능
    this.name = name;     // 원하지 않는 객체 생성 ㅜㅜ
}
```

[TO-BE]

```java
public Name(final String name) {
    this.name = name;     // 객체 생성
    validate(name)      // name == 원하는 값 ? 정상 진행 : 예외 발생
}
```

## 질문
1. 방어적 복사는 어디까지 사용해야 하나?
  - 도메인에서 dto로 방어적 복사 get or dto에서 view로 방어적 복사 get
  - 혹은 둘 다?
2. 싱글 스레드 환경에서 멀티 스레딩으로 확장까지 고려해야 하나?
