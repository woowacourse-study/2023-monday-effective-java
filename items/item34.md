### 가장 단순한 형태의 enum
```java
public enum Number {
    ACE, TWO, THREE, FOUR, FIVE, SIX, SEVEN,
    EIGHT, NINE, TEN, JACK, QUEEN, KING;
}
```

- 위와 같이 사용하면 단순한 상수의 집합이 된다.
- `public static final`로 상수화 시키면 되는데 굳이 사용하는 이유가 무엇일까?

<br>

1. enum은 완전한 형태의 class이다.
   - 즉, 상수의 저장 기능 말고 객체답게 동작할 수 있도록 한다는 뜻이다.
   - 예를 들어 상수의 이름을 가지고 오는 메소드를 만들 수 있다. 
```java
public enum Number {
    ACE("A", 1);

    private final String name;
    private final int value;

    Number(String name, int value) {
        this.name = name;
        this.value = value;
    }

    public String getName() {
        return name;
    }
}
```

<br>

2. enum으로 만들어진 인스턴스들은 하나씩만 존재한다.
   - 기본적으로 싱글턴이라고 한다.
   - `equlas()`를 오버라이드 하지 않아도, 같은 인스턴스이기 때문에 true가 나온다.
```java
public boolean isAce() {
    return number.equals(Number.ACE);
}
```

<br>

### 조건
- 모든 필드가 불변이어야 한다. (왜일까?)
  - enum은 불변 클래스이다.
  - 밑과 같은 방식
```java
public final class Point {
    private final int point;
    
    public Point(int point) {
        this.point = point;
    }
    
    public int add(int number) {
        return new Point(point + number);
    }
}
```
- 생성자에서 자신의 인스턴스를 Map과 같은 collection에 추가할 수 없다.
- 생성자에서 다른 인스턴스에도 접근 못한다.
  - 이때는 static 필드들이 초기화 되기 전이기 때문
```java
public enum Number {
    ACE("A", 1),
    TWO("2", 2);

    private final String name;
    private final int value;

    Number(String name, int value) {
        this.name = name;
        this.value = value;
        Number[] numbers = values(); // 이런거 못함
        valueOf("ACE"); // 이런거 못함
    }

    public String getName() {
        return name;
    }
}
```
첫번째 꺼 에러 : Caused by: java.lang.NullPointerException: Cannot invoke "[Lblackjack.domain.card.Number;.clone()" because "blackjack.domain.card.Number.$VALUES" is null
두번째 꺼 에러 : Caused by: java.lang.IllegalArgumentException: blackjack.domain.card.Number is not an enum type

<br>

### 상수별 메소드
- 저희 미션에서 적용해볼만한 부분을 찾지 못해 이렇게 책의 예시를 들고왔습니다.
- 가장 간단한 형태
```java
public enum Operation {
    PLUS {
        public int apply(int x, int y) {return x + y;}
    },
    MINUS {
        public int apply(int x, int y) {return x - y;}
    };
    
    public abstract int apply(int x, int y);
}
```

- 전략 열거 타입
  - 밑의 코드와 같이 enum안에 enum을 정의해준 형태입니다.
  - 그런데 PayType을 밖으로 빼는게 가독성 면에서 좋아보입니다. 여러분들의 생각은 어떠신가요?
```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    // 이부분!
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
