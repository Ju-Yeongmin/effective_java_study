# Item 85
## 자바 직렬화의 대안을 찾으라

### 직렬화란?
1997년, 자바에 직렬화가 처음 도입되었다.
직렬화는 넓은 의미로 직렬화는 어떤 데이터를 다른 데이터의 형태로 변환하는 것을 의미한다
- 자바에서의 직렬화(Serializable)란 바이트 스트림으로의 직렬화로, 객체의 상태를 바이트 스트림으로 변환하는 것을 의미한다
- (반대로 바이트 스트림에서 객체로의 상태 변환은 역직렬화(Deserializable) 라고 함)
- 객체를 바이트 스트림으로 변경하여 다른 환경에서도 사용할 수 있도록 하는 것
- 바이트로 변환하는 이유는 컴퓨터에서 기본으로 처리되는 최소 단위가 바이트이기 때문이다
- (엄밀히 따지만 최소단위는 비트이지만 표현방법이 0과 1뿐이라 너무 적다)
- 쉽게 말하면 출발지와 목적지에서 모두 알아들을 수 있는 바이트로 소통할 수 있도록 한다

그렇다면 직렬화는 어떻게 구현하는데?
```java
// 간단하게 Serializable 인터페이스만 상속하면 바로 직렬화 가능
// 참고로 Serializable은 마커 인터페이스기 때문에 따로 구현할 것은 없음
public class TestClass implements Serializable {
  private final String name;

  public TestClass(String name) {
    this.age = age;
  }
}
```

### 직렬화가 왜 문제가 될까?
직렬화의 근본적인 문제는 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기가 어렵다(보안)
또한, 직렬화를 하고 역직렬화를 할 때 문제가 발생한다
1. 객체를 읽는 READoBJECT 메서드는 클래스 패스에 존재하는 거의 모든 타입의 객체를 만들 수 있다
   - 반환 타입이 Object
2. 바이트 스트림을 역직렬화 하는 과정에서 해당 타입 안의 모든 코드를 수행할 수 있다
   - 객체를 아예 불러올 수 있으므로 모든 코드를 수행할 수 있다
3. 그렇기에 타입 전체가 전부 공격 범위에 들어간다
4. 용량도 다른 포맷에 비해서 몇 배 이상의 크기를 가진다
5. 역질렬화 폭탄을 맞을 수 있다

바이트 스트림으로 직렬화하는데는 시간이 별로 걸리지 않지만 역질렬화를 잘못하면
안으로 들어가서 HashCode 메서드를 계속 호출해야하기 때문에 시간이 오래걸린다

```java
@DisplayName("역직렬화 폭탄 테스트")
@Test
void deserializeBomb() {
    byte[] bomb = bomb();

    // 역직렬화를 하면 엄청 많은 시간이 걸린다.
    deserialize(bomb);
    assertThat(bomb).isNotEmpty();
}

static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1);
        s1.add(t2);
        s2.add(t1);
        s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // 직렬화 수행
}
```
![image](https://github.com/Ju-Yeongmin/effective_java_study/assets/110506500/b7af373f-315c-44c3-92e1-5ed0a7a0491e)


### 가장 좋은 방법은 아무것도 직렬화하지 않는 것
직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 java 9에 나온
ObjectInputFilter를 사용하는 것도 방법이다. 이는 데이터 스트림이 역직렬화되기 전에 필터를
적용해서 특정 클래스를 받거나 거부할 수 있다.

```
핵심!

직렬화는 위험하니 피해야 한다. 신뢰할 수 없는 데이터는 역직렬화하지 말자.
꼭 해야한다면 역직렬화 필터링을 사용하고, 이것도 모든 공격을 막을 수는 없다.
```
