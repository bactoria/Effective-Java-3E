
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

### 상속 vs 컴포지션

**상속**

```java
public class MySet<E> extends HashSet<E> {

}

MyHashSet<String> s = new MyHashSet<>();

```

기존 클래스를 확장

구현 상속(클래스가 클래스를 확장)은 캡슐화를 깨트린다.

상위클래스가 변경되면 하위클래스에 이상이 생길 수 있음.

상속은 is-a 관계일 때만 써야 한다. ex) 소고기는 고기, 귤은 과일

&nbsp;

**컴포지션**

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    //...
}


public class WrapperSet<E> extends ForwardingSet<E> {

}

```

상속대신 컴포지션을 사용하면 됨.

컴포지션 : 기존 클래스를 확장하지 않고, 새로운 클래스(ForwardingSet)를 만들고 private필드로 기존 클래스의 인스턴스(s)를 참조

상위클래스의 결함을 숨길 수 있음.

&nbsp;
&nbsp;

### 상속 (구현상속)

상속용 클래스를 만들려면 클래스를 어떻게 사용하는지 문서로 남기고,

`public/protected 메서드`들을 재정의할 경우 

동작에 어떤 영향을 주는지를 남겨야 함.

문서의 규약을 꼭 지켜야 함.

&nbsp;

상속용으로 설계된 클래스가 아니라면 상속하지 않는 것이 안전.

&nbsp;

클래스 상속 금지 : 1. final 선언  2. static factory

&nbsp;

`Set, List, Map` 의 경우 핵심 기능을 인터페이스가 정의하고, 클래스가 구현했다. 

상속이 필요가 없었다.

&nbsp;
&nbsp;

### 추상클래스 vs 인터페이스


둘 다 인스턴스 생성을 할 수 없고 상속받는 자녀클래스에서 인스턴스 생성이 가능하다.

`추상클래스` : 추상메서드가 하나라도 있는 클래스 (구현된 메소드가 존재할 수 있음)

`인터페이스` : 자바8 이후로 `default method`가 구현이 가능해졌다.

(defalut method의 예로는 Collection 인터페이스의 removeIf()가 있다.)

&nbsp;

다중 구현용으로는 인터페이스가 적절. 

골격 구현도 인터페이스의 `default method` 사용하되 제약이 따를 경우 추상클래스를 사용.

&nbsp;
&nbsp;

### 인터페이스

인터페이스를 사용할 때, 구현하는 쪽을 생각해야 한다.

특히 기존의 인터페이스에 default method를 추가한다고 한다면

구현하는 곳에서 정상적으로 작동해야 한다.

&nbsp;

#### 상수 인터페이스 쓰지말것.

``` 
// 코드 22-1
public interface PhysicalConstants {
    
    //아보가드로 수
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

} 

```

본래 인터페이스는 사용자에게 해당 메서드를 접근할 수 있게 하고, 

내부 구현은 클래스에서 감춘다.

위 상수는 클래스 내부에서 사용되며 사용자가 알 필요가 없는데 사용자가 접근 가능하게 되었다.


TODO :: item23