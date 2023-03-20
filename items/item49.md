# 왜 이 주제를 선택하게 되었는가?
## 의문
1. private 메서드에서 매개변수가 유효한지 검사해야 할까?
2. 현재 사용하지는 않으나 나중에 사용하는 변수는 언제 검사해야 할까?

## 의문의 계기
체스판에 존재한는 두 칸(Square)의 사이에 존재하는 모든 칸을 구하는 기능을 구현했다.

```java
public List<Square> squaresOfPath(Square to) {
        if (inLine(to)) {
            return squaresOfLine(to);
        }
        if (inDiagonal(to)) {
            return squaresOfDiagonal(to);
        }
        return Collections.EMPTY_LIST;
    }
```

```java
private List<Square> squaresOfLine(final Square to) {
        if (!inLine(to)) {
            throw new IllegalArgumentException("직선이 아닙니다");
        }
        if (to.rank == this.rank) {
            return File.filesBetween(this.file, to.file)
                       .stream()
                       .map(foundFile -> Square.of(rank, foundFile))
                       .collect(Collectors.toUnmodifiableList());
        }
        return Rank.ranksBetween(this.rank, to.rank)
                   .stream()
                   .map(foundRank -> Square.of(foundRank, file))
                   .collect(Collectors.toUnmodifiableList());
    }
```

이때 squaresOfLine 메서드에서 직선에 관한 유효성 검사를 해야할까?

위의 메서드는 직선을 이루지 않는 칸이 주어지더라도 터지지 않는다.
다만, 예상치 못한 결과를 낳을 뿐..

### 해야한다!
- private 메서드라도 오용의 여지를 제거하는 것이 좋다.
- squaresOfLine은 잘못된 동작을 하더라도 그 문제를 알아차리기 어렵다. 

### 할 필요 없다!
- 해당 코드를 사용하는 코드에서 이미 유효성 검사를 하고 있다.
- private 메서드는 사용처가 제한되기 때문에 유효성을 보증할 수 있다.

-> 책에서는 private 메서드의 매개변수 검사에 assert라는 문법을 사용하라고 한다.
- private 메서드는 프로그래머에 의해 유효한 값만 들어올 것을 보장받아야 한다.
- private 메서드의 잘못된 매개변수는 고로 예외를 터뜨리기보단 버그로 인식되어야 한다.
- assert는 특정한 옵션을 달아줘야 런타임에 영향을 미친다. 고로 개발 환경에서만 실행시킬 때 사용된다.

```java
private List<Square> squaresOfLine(final Square to){
    assert inLine(to);
        ...
}
```

## 유효성 검사의 원칙
1. 오류는 최대한 빨리 터뜨려야 한다.
- 나중에 터지면 추적하기 어려워진다.
- 나중에 터지면 모호한 예외를 터뜨릴 가능성이 있다.

```java
int pop() {
    return arr[--size]    
}
```
IllegalStateException을 터뜨리는 것이 합당하지만,
적절한 검사가 이루어지지 못해 IndexOutOfBound를 터뜨린다. 

- 프로그램이 전혀 예상치 못한 결과를 만들 수도 있다.</br>
위의 코드에서 size를 감소시키게 되어 다음 번 호출 때도 해당 객체는 사용하지 못하게 된다.

2. 나중에 쓰려고 하는 매개변수의 유효성은 미리 검사하라.
- 매개변수의 유효성을 한 곳에서 관리해야 식별하기 쉽다.
- 생성자에서 항상 유효성 검사를 하는 이유가 이 때문이다. 그렇게 함으로써 불안정한 객체의 생성을 막을 수 있다.

### 그렇다면 항상 메서드 바디가 실행되기 전에 유효성 검사를 해야할까?
아래와 같은 경우에는 하지 않아도 괜찮다.
1. 유효성 검사가 실용적이지 않을 때
2. 계산 과정에서 암묵적으로 계산될 때
- Collections.sort를 보면 매개변수의 유효성 검사를 하지 않는다.
- Collections.sort는 계산 과정 중에 타입이 일치하지 않으면 예외를 던지기 때문이다.

단, 실패 원자성이 깨질 수 있기 때문에 조심해야 한다.

이때 실패 원자성이 뭘까?

# item 76
메서드 실행이 실패하더라도 객체의 상태가 안정적인 것을 실패 원자적이라고 말한다.  

실패원자적이라면 에러가 발생하더라도 그 객체를 계속 사용할 수 있다.

실패 원자성을 만드는 방법
1. 불변 객체를 사용한다. 불변 객체는 본질적으로 실패 원자적이다.
2. 코드의 계산부가 실행되기 전에 매개변수를 검사한다.
```java
int pop() {
    if (size == 0) throw new EmptyStackException();
    Object ret = arr[--size]
    arr[size] = null;
    return ret;
} 
```
3. 코드의 임시 복사본을 사용하여 작업을 수행하고 작업 수행이 끝나고 객체의 상태를 변경한다.
