# [Item 45] 스트림은 주의해서 사용하라

- - - -

## 스트림이 추상화하는 핵심 개념

### 1. 무한한 값이 올 수 있는 순서가 있는 데이터의 모음

스트림은 lazy한 연산을 지원하기에 무한한 값을 표현할 수 있다.<br>
스트림은 최종연산이 오기 전까지 어떠한 동작도 하지 않는다.

```java
List<String> names=List.of("홍실","에단","디투","오리","에밀","모디");
final Stream<String> stringStream=names.stream()
        .map(name->{
            System.out.println(name);
            return"스터디 참여자"+name;
        });
출력값 없음
```

최종연산이 오기전이기에,위 코드를 실행하더라도 아무런 값도 출력하지 않는다.<br>
스트림은 **최종연산이 오기전까지 함수를 저장하는 느낌의 역할**을 한다.<br><br>
그렇기에 무한스트림이 있을 수도 있다. 사실 여기서 무한 스트림이 가능하다는 사실이 중요한 것이 아니라, <br>
**최종연산이 오기전까지 연산이 지연된다는 사실이 중요하다.**

### 2. 파이프라인은 원소들로 수행하는 연산 단계를 표현하는 개념

스트림 파이프 라인은 데이터 소스를 여러 중간 연산과 하나의 종단 연산으로 구성된다.<br>
값을 가공하거나(map), 일부를 제거하거나(filter), 정렬하거나(sorted)와 같은 내부의 값에 변화를 주는 메서드들이 있고<br>
요소들을 하나로 리듀싱하거나(reduce), 새로운 collection으로 만들어 반환하거나(collect), <br>
요소들의 수를 세는(count)등 마지막 값을 반환하는 마무리 연산이 있습니다.<br>
스트림의 메서드들은 위와 같은 연산들을 제공 하므로 가독성이 좋습니다.

## 스트림 + 람다의 장단점

### 스트림 + 람다로 할 수 없는 일들

- 지역 변수를 수정하는 일 : <br>
  스트림 안의 람다는 스트림 밖의 변수를 수정하지 못 한다.<br>
  여기서 수정하지 못한다는 말은 대입이 불가능하단 이야기다.<br>
  (jcf의 add, remove 메서드는 호출이 가능하다.) <br>
- 중간에 return 또는 break로 반복의 흐름을 조절할 수 없다 또는 CheckedException을 던질 수 없다.<br>
  -> 사실 stream.takeWhile(Predicate<T>)를 이용해 break 와 같은 연산을 할 수 있다.

### 스트림 + 람다로 할 수 있는 일들

- 원소들을 mapping 한다. (Stream.map)
- 원소들을 필터링한다. (Stream.filter)
- 원소들을 하나의 연산을 사용해 결합한다. (더하기, 연결하기, 최솟값 구하기) <br>
  (IntStream.sum, Stream.reduce, IntStream.min)
- 원소들을 컬렉션에 모은다. (Stream.Collect)
- 원소들에서 특정 조건을 만족하는 원소를 찾는다. (Stream.findAny)
- 병렬 연산이 매우 쉬워진다. 단순히 Stream을 parrallelStream으로 생성하기만 하면 병렬 연산이 된다.

## 실제 비교

- stream 을 사용했을 때

```java
private static final List<Card> baseDeck;

static {
        baseDeck=Arrays.stream(CardShape.values())
            .map(RandomDeckGenerator::generateCardsFrom)
            .flatMap(List::stream)
            .collect(Collectors.toUnmodifiableList());
}

private static List<Card> generateCardsFrom(final CardShape shape){
        return Arrays.stream(CardNumber.values())
            .map(cardNumber->new Card(shape,cardNumber))
            .collect(Collectors.toUnmodifiableList());
}
```

- for-loop 를 사용했을 때

```java
private static final List<Card> baseDeck;

static {
        baseDeck=new ArrayList<>();
        for(final CardShape shape:CardShape.values()){
            for(final CardNumber number:CardNumber.values()){
                baseDeck.add(new Card(shape,number));
            }
        }
}

```

어느쪽이 더 낫다고 생각하나요. <br>
flatMap이 익숙하시면, stream을 사용하신게, 아니라면 for-loop가 더 낫다고 느끼실 겁니다.<br>

## 최종 결론

### 책에서 나온 최종 결론

스트림이 나은 경우도 있고, for-loop가 알맞은 방식도 있다.<br>
두 방식을 조합하는 방향이 가장 적절하다.<br>
어느쪽이 나온지 확연히 들어나는 경우도 많지만(스트림이 할 수 없는 연산) <br>
아닌 경우에(스트림이나 for-loop 둘 다 가능한 경우)는 아래와 같이 판단해라. <br>
**스트림과 반복 중 어느쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택하라....**

### 개인적인 결론

하지만 뭔가 여기서 마무리 하긴 아쉬웠습니다. 하여, Item 46까지 봐서 정리를 해봤습니다.

# [Item 46] 스트림에서는 부작용(사이드 이펙트) 없는 함수를 사용하라

스트림에서의 핵심은 중간연산을 통해서 하나의 결과로 반환하는 것입니다. <br>
여기서 가장 중요한 부분은 다음과 같습니다. **각 변환 단계는 순수함수여야 합니다.** <br>
(순수함수 : 어떤 함수에 동일한 인자를 주었을 때 항상 같은 값을 리턴하는 함수)<br>
예를 들어 다음과 같은 함수는 순수함수가 아닙니다. <br>

아래는 처음에 공개할 카드 그룹을 Map형태로 반환하는 코드입니다.

```java
stream foreach
final List<Player> players
final Map<Name, CardGroup> firstOpenCardGroup=new LinkedHashMap<>();
        players.stream()
            .forEach(player->firstOpenCardGroup.put(player.getName(),player.getFirstOpenCardGroup()));
        return Collections.unmodifiableMap(firstOpenCardGroup);
```

위와 같은 연산은 아무런 문제가 없습니다. 의도한 결과를 내겠죠<br>
하지만 전혀 스트림 답지 못합니다. 스트림 코드를 가장한 for-loop죠.<br>
오히려 stream을 생성하고 파이프라인을 구성하는 비용이 더 많이 들어 속도도 느릴 것입니다.<br>

```java
Collections.forEach
final List<Player> plyaers;
final Map<Name, CardGroup> firstOpenCardGroup=new LinkedHashMap<>();
        players.forEach(player->
            firstOpenCardGroup.put(player.getName(),player.getFirstOpenCardGroup()));
        return Collections.unmodifiableMap(firstOpenCardGroup);
```

아 추가적으로 Collections.forEach와 Stream.forEach는 다릅니다.

```java
Collections.forEach
default void forEach(Consumer<? super T>action){
        Objects.requireNonNull(action);
        for(T t:this){
            action.accept(t);
        }
}
일반적인 for-loop와 동일하다.
```

```java
@Override
public void forEach(IntConsumer action){
        if(!isParallel()){
            adapt(sourceStageSpliterator()).forEachRemaining(action);
        }
        else{
            super.forEach(action); 
        }
}
그만 알아보도록 하자
```
그만 알아보도록 하자. 원래 forEach가 어떻게 동작하는지 적을라 햇는데, 너무 방대해서 패스<br>
기회가 되면 파이프라인이 생성되는 것을 학습로그에 정리해보도록 하겠습니다.

잠깐 다른 길로 샜는데, Stream을 사용하기 좋은 경우는 <br>
기존에 있는 데이터 소스들로 유의미한 값을 구하는 경우입니다. 스트림은 그런 면에서 강력한 연산들을 제공합니다.<br>

1. Collect연산으로 새로운 컬렉션을 반환 
    1. GroupingBy(Function<T,K>) : Function의 반환값을 key로 갖고, 기존 멤버들을 value로 갖는 map 반환
    ```java
    public Map<WinningStatus, Long> getDealerWinningResults() {
    return getPlayersWinningResults()
        .values()
        .stream()
        .collect(collectingAndThen(groupingBy(WinningStatus::opposite, counting()),
            Collections::unmodifiableMap));
        }
    //winingStatus::opposite은 반대값을 반환하는 메서드
    ```
    위 코드는 player들의 승,무,패로 딜러의 승,무,패를 구하는 코드입니다. 
    2. toList, toMap, toSet : Collection으로 반환
2. sum, maxBy, minBy, count 등 하나의 값을 반환

즉, Stream은 하나의 결과를 반환하기 위한 연산으로서 사용되어야지, 반복문처럼 사용해서는 안된다.<br>
