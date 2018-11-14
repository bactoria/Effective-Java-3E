
## 2장. 객체 생성과 파괴

>객체를 만들어야할 때와 만들지 말아야할 때를 구분하기
올바른 객체 생성하기
객체를 올바르게 파괴하는법

&nbsp;

### static method

=> 팩터리 메서드 (디자인패턴의 팩터리 메서드와는 전혀 다른 내용임.)

&nbsp;

**Boolean.class**
```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean> {

    //static field
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    //static method
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }

    //...
}
```

`static` 은 객체를 생성하지 않아도 사용 가능.

유틸리티성 메서드들은 `static method`로 설계하는 것이 이득임.

위의 **valueOf(boolean b)** 는 Boolean 객체를 생성하지 않아도 사용 가능함.

`ex) System.out.println(java.lang.String.valueOf(true));`

&nbsp;

**User.class**
```java
public class User() {
    private String name;

    private User(name) {
        this.name = name;
    }

    public static User createUser(String name) {
        return new User(name)
    }
}
```

어? new Boolean(true) 니까 TRUE를 만들 때마다 new Boolean 을 생성하는 것이 아니냐고 할 수 있는데 나도 잠시 헷갈렸음. 

이거 TRUE는 프로그램종료될때까지 계속 같은 Boolean 객체를 참조함. 

TRUE를 계속 만들면 그것들은 같은 객체를 참조하고 있는 것임.

이렇게 쓰려면 new Boolean(true) 가 불변이어야 함.

=> **public 생성자 없이도 객체 생성 가능.**

( static method 의 장단점 : item1 참고)

&nbsp;
&nbsp;

### Builder 패턴

**User.class**
```java
public class User() {
    private Long id;
    private String name;
    private String password;

    @Builder
    private User(Long id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }
}
```

`ex) User.builder().id(1L).name("황준오").build();`

`ex) User.builder().id(2L).name("황준오").password("pass").build();`

`builder()` 가 `static method` 이다.

&nbsp;

**빌더패턴은 계층형 클래스에서 쓰기 좋다**

- 피자 (`public abstract class Pizza`) 코드 2-4
    + 뉴욕 피자 (`public class NyPizza extends Pizza`) 코드 2-5
    + 칼초네 피자 (`public class Calzone extends Pizza`) 코드 2-6

성능상으로는 매개변수가 4개 이상일 때 값어치를 한다.

근데 빌더 생성 비용이 크진 않음.

&nbsp;
&nbsp;

### Singleton

#### 싱글톤 1. static final 변수

```java
public class Util {
    public static final Util INSTANCE = new Util();

    private Util(){
    }

    //...
}
```

**문제점 1) 리플렉션으로 private 생성자로 객체생성 가능.**
```java
        Constructor<?> con = Util.class.getDeclaredConstructors()[0];
        con.setAccessible(true);

        Util util1 = (Util) con.newInstance();
        Util util2 = Util.INSTANCE;

        System.out.println(util1); //Util@4554617c
        System.out.println(util2); //Util@74a14482   hashcode가 다르다!!
```

**문제점 2) 직렬화 할 때 추가적인 작업 필요**

implements Serializable 만으로 직렬화를 하면

역직렬화할 때마다 새로운 인스턴스가 만들어짐.

&nbsp;

#### 싱글톤 2. staic method

```java
public class Util {
    private static final Util INSTANCE = new Util();

    public static Util getInstance() {
        return INSTANCE;
    }

    //...
}
```

**문제점 1) 리플렉션으로 private 생성자로 객체생성 가능.**
```java
        Constructor<?> con = Util.class.getDeclaredConstructors()[0];
        con.setAccessible(true);

        Util util1 = (Util) con.newInstance();
        Util util2 = Util.getInstance();

        System.out.println(util1); //Util@4554617c
        System.out.println(util2); //Util@74a14482   hashcode가 다르다!!
```

**문제점 2) 직렬화 할 때 추가적인 작업 필요**

implements Serializable 만으로 직렬화를 하면

역직렬화할 때마다 새로운 인스턴스가 만들어짐.

&nbsp;

#### 싱글톤 3. enum

```java
// 열거 타입 방식의 싱글턴 - 바람직한 방법 (25쪽)
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}

https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item3/enumtype/Elvis.java
```

**직렬화, 리플렉션 에도 싱글톤 유지함.**

&nbsp;
&nbsp;

### 유틸리티 클래스의 생성자

**java.lang.Math**
```java
public final class Math {

    // 생성자가 존재하는데 호출이 불가능한 것은 직관적이지 않으므로 아래처럼 주석을 다는 것이 좋음.
    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}
```

`Math math = new Math();` 는 에러다.

왜? 애초에 Math는 유틸리티성으로 만들었다.

안에 보면 필드,메서드 모두 **static** 이다.

&nbsp;

**`item4`에서 말하고자하는 것은 이런 유틸리티 클래스(Math, Arrays, Collections 등)에 private 생성자를 만들라는 것이다.**

만들지 않으면 public Math()가 생성되므로 실수로 객체를 생성할 여지가 있으니까.

&nbsp;
&nbsp;


