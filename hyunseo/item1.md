
# Effective Java  - Item 1 

# 생성자 대신 정적 팩토리 메소드를 고려하라.


+@ 생성자에 로직이 들어가는것이 좋은가


- ### 정적 팩토리 메소드(static factory method)란?
    
    - ### 일반적으로 객체 생성시 사용하게 되는 생성자 대신 static method를 호출하여 객체를 생성하도록 하는 패턴


- **생성자를 통해 객체를 생성하는 코드**
```
public Car(Name name) {
    this.name = name;
}
```


- **정적 팩토리 메소드 활용**
```
private Car(Name name) {
    this.name = name;
}

public static Car from(Name name) {
    return new Car(name);
}
```

***

### 장점 1. **객체 생성 로직을 캡슐화 할 수 있다.**
3. **객체 생성시 캐싱이 가능하다(매번 새로운 객체를 생성하지 않다도 된다.)**

- **정적 팩토리 메소드를 통해, 도메인 객체를 DTO화 시키는 로직을 캡슐화**
- 적용 전
```
public WinnersNameDto findWinners(Cars cars) {
    List<Car> winners = cars.findWinners();
    return winners.stream()
            .map(Car::getName)
            .collect(collectingAndThen(toList(), WinnersNameDto::new));
}
```

- 적용 후
```
public WinnersNameDto findWinners(Cars cars) {
    List<Car> winners = cars.findWinners();
    return WinnersNameDto.from(winners);
}
    
    ....
    
public class WinnersNameDto {

    private final List<String> winnersNames;

    private WinnersNameDto(List<String> winnersNames) {
        this.winnersNames = winnersNames;
    }

    public static WinnersNameDto from(List<Car> winningCars) {
        return winningCars.stream()
                .map(Car::getName)
                .collect(collectingAndThen(toList(), WinnersNameDto::new));
    }

```

물론 정적 팩토리 메소드를 활용하는게 아니라, 단순히 생성자에 ```List<Car>```를 인자를 넘겨서 생성자 내에서 이러한 작업을 처리할 수도 있다.

그러나 일반적으로 코드를 읽는 사람은, 생성자를 통해 들어간 인자가 클래스의 단순히 초기화할 것이라고 생각하는 경우가 많다.

그런데, 만약 정적 팩토리 메소드를 사용한다면, 일반적인 생성자와는 다르게 어떤 로직이 들어갔을 것이라고 사람들이 추측할 수 있다.


### 장점 2. **메소드에 이름을 정할 수 있다.**

  - 또한 여러개의 정적 팩토리 메소드를 두어서, 특별한 값을 할당할 수 있다.

```
  public Position(int position) {
      this.position = position;
  }

  public static Position create(int position) {
      return new Position(position);
  }

  public static Position createStartPosition() {
      return new Position(1);
  }
```


### 장점 3. 호출 할 때마다 새로운 객체를 생성할 필요가 없다(캐싱 가능).

```
public class LottoNumber {
  private static final int MIN_LOTTO_NUMBER = 1;
  private static final int MAX_LOTTO_NUMBER = 45;

  private static Map<Integer, LottoNumber> lottoNumberCache = new HashMap<>();

  static {
    IntStream.range(MIN_LOTTO_NUMBER, MAX_LOTTO_NUMBER)
                .forEach(i -> lottoNumberCache.put(i, new LottoNumber(i)));
  }

  private int number;

  private LottoNumber(int number) {
    this.number = number;
  }

  public LottoNumber of(int number) {  // LottoNumber를 반환하는 정적 팩토리 메서드
    return lottoNumberCache.get(number);
  }

  ...
}
```

매번 특정한 number의 LottoNumber를 생성할 필요가 없다.

대부분의 경우 이런 부분은 enum으로 가능하지만, 때때로 정적 팩토리 메소드를 해결해야 할 때가 있음.


### 장점 4. 하위 자료형 객체를 반환할 수 있다.
  - 생성자와는 다르게 정적 팩토리 메소드는 반환값을 가지고 있기 때문에 상속받은 하위 자료형 객체를 반환할 수 있다.
```
public class Level {
  ...
  public static Level of(int score) {
    if (score < 50) {
      return new Basic();
    } else if (score < 80) {
      return new Intermediate();
    } else {
      return new Advanced();
    }
  }
  ...
}
```
***
### 단점

1. 생성자가 private만 있고 정적 팩토리 메소드를 통해서만 생성하면 상속이 불가능함.
   (상속하면 필연적으로, 상위 객체의 생성자를 호출해야하는데 호출이 불가능)
2. 정적 팩토리 메소드를 통해서만 객체를 생성하는 클래스가, 코드가 길어진다면, 사람들이 이 객체를 어떻게 생성하는지 파악하기 어렵다.
   (물론 주석을 통해, 이를 설명해줄 수 있지만, 유지 보수해야할 포인트가 늘어난다.)

   

***


다만 객체의 생성할 때, 로직을 캡슐화하고 이에 대한 네이밍을 주기 위해 **정적 팩토리 메소드**를 사용한다면, 이때 객체 생성에 너무 많은 로직이
들어간 것 아닐까(어쩌면 책임의 분리가 제대로 안됐을 수도 있다.) 고려해 볼 필요가 있다.  



생성자에 너무 많은 로직이 들어간다면, 무거워진다.(객체 생성이 어려워진다.) 이렇게 된다면...

- 테스트 하기 어렵다
- 단일 책임 원칙이 제대로 안지켜졌을 수도 있음.

객체 생성에 너무 많은 책임이 주어지다보니...
```
private Ladder(List<Line> lines, LadderHeight ladderHeight) {
    this.lines = new ArrayList<>(lines);
    this.ladderHeight = ladderHeight;
}

public static Ladder create(int numberOfPeople,
                            LadderHeight ladderHeight,
                            NumberGenerator numberGenerator) {
    List<Line> lines = new ArrayList<>();
    int width = numberOfPeople - NUMBER_OF_PEOPLE_TO_WIDTH_SCALE;

    for (int i = 0; i < ladderHeight.getLadderHeight(); i++) {
        lines.add(Line.create(numberGenerator, width));
    }

    return new Ladder(lines, ladderHeight);
}
    
    .
    .
    .

테스트 시 객체를 원하는 형식으로 만들기도 어렵고 불필요한 의존이 생기거나 때때론 로직이 복잡해진다.....
|
|
v

@DisplayName("다리가 높이와 라인 수를 입력받고, 각 라인들에 높이만큼의 크기를 가진 Point 리스트를 생성한다")
@Test
void create_success() {
    LadderHeight ladderHeight = new LadderHeight(3);
    Ladder ladder = Ladder.create(3, ladderHeight, new RandomNumberGenerator());
    .
    .
    .
```


Ladder 생성의 책임을 다른 클래스에 이관했더니..
```
public class LadderGenerator {

    private static final int NUMBER_OF_PEOPLE_TO_WIDTH_SCALE = 1;

    private final LineGenerator lineGenerator;

    public LadderGenerator(LineGenerator lineGenerator) {
        this.lineGenerator = lineGenerator;
    }

    public Ladder generate(int numberOfPeople,
                           LadderHeight ladderHeight) {
        List<Line> lines = new ArrayList<>();
        int width = numberOfPeople - NUMBER_OF_PEOPLE_TO_WIDTH_SCALE;
        int height = ladderHeight.getLadderHeight();

        while (height-- > 0) {
            lines.add(lineGenerator.generate(width));
        }
        return new Ladder(lines);
    }
}
```


테스트에 사용할 Ladder 객체를 내가 원하는대로 만들기 쉬워졌다..

```
    /**
     * pobi  neo   me    ohs   hello
     * |-----|     |-----|     |
     * |     |-----|     |-----|
     * |     |-----|     |-----|
     * |     |-----|     |-----|
     */
    @DisplayName("move()를 통해 게임 결과에 맞는 positon을 반환한다.")
    @ParameterizedTest
    @CsvSource(value = {"1:3", "2:1", "3:5", "4:2", "5:4"}, delimiter = ':')
    void move_result_test(int startPosition, int expectedPosition) {
        // given
        Ladder ladder = new Ladder(createLine());
        Player player = new Player(new Name("hs"), new Position(startPosition));
        // when
        Position actualPosition = player.move(ladder);
        // then
        assertThat(actualPosition).isEqualTo(new Position(expectedPosition));
    }

    private List<Line> createLine() {
        return List.of(
                new Line(List.of(PASSABLE, BLOCKED, PASSABLE, BLOCKED)),
                new Line(List.of(BLOCKED, PASSABLE, BLOCKED, PASSABLE)),
                new Line(List.of(BLOCKED, PASSABLE, BLOCKED, PASSABLE)),
                new Line(List.of(BLOCKED, PASSABLE, BLOCKED, PASSABLE)));
    }
```

![image](img.PNG)

