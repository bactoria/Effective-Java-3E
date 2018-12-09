
## 4장. 클래스와 인터페이스

> Class & Interface

&nbsp;

- 클래스와 멤버의 접근 권한 최소화
    + 클래스
        * private
        * protected
        * public
    + 멤버
        * private
        * package-private
        * protected
        * public (public static final 필드 외에는 public으로 하지 말 것)

가능한 접근권한을 최소화 할 것.

테스트를 실행시키기 위해 클래스를 public 으로 수정하는 짓은 절대 하지 말 것.

**public static final Object[]**
```java
// Bad! 불변식이 아님. 외부에서 Thing을 마음대로 수정할 수 있음.
public static final Thing[] VALUES= {...}; 


// 1. 리스트 (Good!)
private static final Thing[] PRIVATE_VALUES = {...};

public static final List<Thing> VALUES= {
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES))
};


// 2. 방어적 복사 (Good!)
private static final Thing[] PRIVATE_VALUES = {...};

puvlic static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}

```

TODO :: p100,101 모듈얘기.. 

&nbsp;
&nbsp;

### Getter/Setter

**(Non Getter/Setter)**
```
public Class User {
    public String id;
    public String password;
    public String name;
}
```

- 장점
    + 1.클래스 내부 표현방식을 언제든지 바꿀 수 있는 유연성
        * setter 시 입력 방법 바꿀 수 있음
        * getter 시 출력 방식 바꿀 수 있음
    + 2.가변 필드를 방어적 복사 할 수 있음
        * private String[] values;

&nbsp;

위를 어긴 라이브러리 : Point, Dimension

```java
class Point {
    public double x;
    public double y;
}
```

&nbsp;
&nbsp;

### 불변클래스

인스턴스의 내부 값을 수정할 수 없는 클래스

가변클래스보다 설계하기도 쉽고 훨씬 안전하다.

스레드에 안전, 따로 동기화 필요없음.

단점 : 값이 바뀔 때마다 새로 객체 만들어야 함.

ex) String, BigInteger, BigDecimal

&nbsp;

#### 규칙

- setter를 제공하지 않는다.
- 클래스를 상속할 수 없도록 한다.
- 모든 필드를 private final 로 선언한다.
- 가변 객체가 있다면 방어적복사를 수행한다.

&nbsp;
&nbsp;

### Todo :: item 18
