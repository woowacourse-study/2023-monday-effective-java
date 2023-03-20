아이템 18. 상속보다는 컴포지션을 사용하라


상속을 사용했을때 나타날 수 있는 문제점

### 불필요한 인터페이스 상속문제

![image](images/item18_1.PNG)

Vector는 java.util.List의 초기 버전이라고 할 수 있다.

Stack은 가장 나중에 추가된 요소가 가장 먼저 추출되는(Last In First Out, LIFO) 구조이다.

Stack을 구현할때 Vector의 get, add , remove를 재사용하기 위해 상속을 하였다.

다만 Stack은 Vector와 다르게 맨 마지막 위치에서만 요소를 추가하거나 제거해야한다.

```
@Test
void test1() {
    final Stack<String> stack = new Stack<>();
    stack.push("1");
    stack.push("2");
    stack.push("3");

    stack.add(0, "4");

    assertEquals("4", stack.pop());

    //expected: <4> but was: <3>
    //Expected :4
    //Actual   :3
}
```

-> add를 안쓰면 되지만, 이런 설계 자체가 좋지않다.
-> 불필요한 퍼블릭 인터페이스 때문에 상속받은 부모 클래스의 메서드가 자식 클래스의 내부 구조에 대한 규칙을 깨트릴 수 있다.


### 메서드 오버라이딩의 오작용 문제

```
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(final E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(final Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

```
    @Test
    void test1() {
        final InstrumentedHashSet<String> instrumentedHashSet = new InstrumentedHashSet<>();
        instrumentedHashSet.addAll(List.of("1", "2", "3"));

        assertEquals(instrumentedHashSet.getAddCount(), 3);

//        expected: <6> but was: <3>
//        Expected :6
//        Actual   :3
}
```

addAll은 내부적으로 add를 호출하는 구조라 addCount가 2배로 증가한다.

이를 해결할려면 addAll method를 따로 override 안하면 된다.

그런데 만약 HashSet의 addAll이 변경되어 add가 아닌 다른 방법을 사용한다면 이번엔 누락되는 문제가 생길 것이다.

이에 대한 해결법은 addAll을 override한 후 부모에서 그대로 코드를 들고 오는 것이다.

허나 이는 미래에 문제가 발생할 수 있다는 이유로, 코드의 중복을 만들었다는 문제가 발생한다.

또한 소스코드에 대한 접근 권한이 없다거나 addAll이 private method를 호출한다면 단순히 코드를 그대로 가져오는 것도 어렵다.


### 부모 클래스와 자식 클래스의 동시 수정 문제

```
public class PlayList {
    private List<Song> tracks = new ArrayList<>();

    public void append(Song song) {
        tracks.add(song);
    }
}

public class PersonalPlayLIst extends PlayList {

    public void remove(Song song) {
        getTracks().remove(song);
    }
}
```


가수의 이름을 key, 노래의 제목을 value로 가지도록 변경됐다.

```
public class PlayList {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();

    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());
    }
    

public class PersonalPlayLIst extends PlayList {

    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSingler());
    }
}
```

부모 클래스의 코드가 변경되면 자식의 코드가 깨질 수 있다.

-> 클래스를 상속하면 결합도로 인해 자식 클래스와 부모 클래스의 구현을 영원히 변경하지 않거나, 자식 클래스와 부모 클래스를 동시에 변경하거나 
둘 중 하나를 선택할 수 밖에 없다.


## 조합을 통한 해결

### InstrumentedHashSet<E>의 메서드 오버라이딩 오작용 문제 해결
```

public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    private Set<E> set;

    @Override
    public boolean add(final E e) {
        addCount++;
        return set.add(e);
    }

    @Override
    public boolean addAll(final Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```


### 메서드 오버라이딩의 오작용 문제를 조합으로

조합을 사용하여 Stack의 문제점을 해결
-> 불필요한 퍼블릭 인터페이스를 노출하지 않고, 필요한것만 사용할 수 있다.
```
public class Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }

        return elements.remove(elements.size() - 1);
    }
}

```


```
public class PersonalPlayLIst {

    private PlayList playList = new PlayList();

    public void append(Song song) {
        playList.append(song);
    }

    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSingler());
    }
}
```

--> 조합을 사용하더라도 딱히 동시 수정 문제가 달라지진 않았다.
![image](images/item18_2.PNG)



---

중복을 찾아라

중복의 코드를 abstract class 혹은 조합으로 해결할 수 있는 고민하라(is a - has a 등) 

상속을 하더라도 구현이 아닌 추상에 의존하다면 문제를 덜 수 있다.

그리고 최대한 다른것만 남기고 중복만 상위로 올려라.

super method 참조 보다는 abstract 혹은 final 키워드를 통해 문제 발생 가능성을 최소화 할 수 있다.

물론 추상 클래스를 상속 받더라도 어느정도 단점은 존재한다.

새로운 필드가 추가되면 하위의 생성자를 변경해야한다.
-> 하지만 컴파일단에서 잡을 수 있고 세부적인 로직을 변경해야하는건 아니다.

부모의 구현을 알아야해서 캡슐화가 깨지는 부분이 있다.
-> 중복 코드를 제거하는 것에 대한 trade-off

웬만하면 구현체를 상속받진 말자.
