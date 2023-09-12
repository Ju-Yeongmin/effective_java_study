# Item 49
## 과도한 동기화는 피하라
아이템78 에서 충분하지 못한 동기화의 피해를 다뤘다면 이번에는 반대상황을 살펴보자

### 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 낳는다
응답 불가와 안전 실패를 피하려면 동기화 메서드나 블록 안에서는 제어를 절대 클라이언트에 양도하면 안된다
- 동기화된 영역 안에서는 재정의할 수 있는 메서드를 호출하면 안된다
- 클라이언트가 넘겨준 함수객체를 호출해서도 안된다

### 외계인 메서드
응답 불가와 안전 실패를 유발할 수 있는 메서드
- 동기화 영역에서 재정의할 수 있는 메서드,
- 클라이언트가 넘겨준 함수 객체 등을 동기화된 클래스 관점에서 외계인 메서드라고 한다
- 동기화된 클래스는 외계인 메서드가 무슨 일을 할지 알 수 없다
- 외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착 상태에 빠지거나, 데이터를 휀손할 수 있다


```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>(); // 관찰자리스트 보관

    public void addObserver(SetObserver<E> observer) { // 관찰자 추가
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) { // 관찰자제거
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) { // Set에 add하면 관찰자의 added 메서드를 호출한다.
        synchronized (observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```
위 코드는 잘못된 코드 작성의 예로, 동기화 블록 안에서 외계인 메서드를 호출 한다

ObservableSet에 관찰자 SetObserver를 추가하면(addOpserver) ObservableSet.add 메서드가
호출될 때마다 notifyElementAdded가 호출되고, 추가된 관찰자의 Added 메서드가 호출된다(observer.added())
관찰자 패턴을 사용하여 집합에 원소가 추가되면 알림을 받을 수 있다

```java
public class Test1 {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver((set1, element) -> System.out.println(element)); // add 된 원소를 출력한다.
    // 제어로직이 람다식으로 밖에있다

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
위의 메서드를 실행하면 정상적으로 동작하여 0에서 99까지를 출력한다


### 문제를 일으킬 수 있는 익명함수 콜백
```java
public class Test2 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            @Override
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지한다.
                    s.removeObserver(this); //this 를 넘겨주어야하기 때문에 람다로는 안됨.
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
위의 메서드를 실행하면 아래의 오류를 뱉고 실행을 중단한다
![image](https://github.com/Ju-Yeongmin/effective_java_study/assets/110506500/727ba077-7ced-43bf-81c9-3b404a3447cb)

set.add()가 호출되면 notifyElementAdded() 메서드가 호출된다
그리고 이 메서드는 for-each 문으로 각 관찰자를 순회하며 관찰자의 added 메서드를 호출한다

하지만 메인 메서드에서 정의한 added 메서드는 removeObserver 메서드를 호출하여 자기 자신을 넘겨주는데,
이미 for-each 문에서 observers를 synchrosized로 감싸고 있기 때문에 removeObserver 호출 시
ConcurrentModificationException 이 발생한다 
콜백을 거쳐 돌아와서 수정되는 것까지 막지는 못하기 때문이다.

### 문제를 일으키는 스레드 - 교착상태

```java
public class Test3 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

        // 코드 79-2 쓸데없이 백그라운드 스레드를 사용하는 관찰자 (423쪽)
        set.addObserver(new SetObserver<Integer>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec =
                        Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```
위의 exec 는 새로운 스레드에 removeObserver를 수행하는 람다함수를 넘긴다
이 메서드를 실행하면 23까지 출력하고 계속 멈춰있는 상태가 지속된다
removeObserver를 수행하기 위해서는 observers(락)를 획득해야하는데 
observers는 메인스레드에서 실행중인 notifyElementAdded 메서드에 의해 획득될 수 없다


### 불변식이 임시로 깨진 경우
위의 경우는 문제가 생길 수는 있지만 observers의 일관성이 깨지지는 않는다.
락의 재진입 가능성: 이미 락을 획득한 스레드는 다른 synchronized 블록을 만났을 때 락을 다시 검사하지 않고 진입 가능하다.
재진입 가능한 락은 다음과 같이 교착상태를 회피할 수는 있게하지만, 안전실패(데이터훼손)로 변모시킬 수 있다.

```java
public class Test {
  public synchronized void a() {
    b(); // 이론적으로라면 여기서 교착상태여야하지만 같은 스레드에 한해 재진입을 허용하기 때문에 
  }
  public synchronized void b() { // 진입 가능하다
  }
  public static void main(String[] args) {
    new Test().a();
  }
}
```

#### 동기화의 기본 법칙은 동기화 영역 내에서는 가능한 일을 적게하는 것


```java
핵심!
교착상태와 데이터 훼손을 피하려면 동기화 영역안에서 외계인 메서드를 절대 호출하지 말자
일반화하면, 동기화 영역안에서의 작업은 최소한으로 줄이자. 합당한 이유가 있을 때만
내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자
```
