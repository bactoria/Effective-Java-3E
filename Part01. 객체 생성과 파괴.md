
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

    // static field
    // TRUE는 프로그램이 종료될 때까지 같은 Boolean 객체를 참조 (FALSE도)
    public static final Boolean TRUE = new Boolean(true); 
    public static final Boolean FALSE = new Boolean(false);

    // static method (Boolean 객체를 생성하지 않아도 사용 가능)
    // ex) System.out.println(java.lang.String.valueOf(true));
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }

    //...
}
```

`static` 은 객체를 생성하지 않아도 사용 가능.

유틸리티성 메서드들은 `static method`로 설계하는 것이 이득임.

&nbsp;

`TRUE`를 계속 만들면 그것들은 같은 객체를 참조하고 있는 것임.

이렇게 쓰려면 `new Boolean(true)` 가 불변이어야 함.

&nbsp;

**User.class**
```java
// public 생성자 없이 객체 생성이 가능
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

( static method 의 장단점 : item1 참고)

&nbsp;
&nbsp;

### Builder 패턴

**User.class**
```java
import lombok.Builder;

class User {
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

public class Main {
    public static void main(String[] args) {

        //builder()가 static method임
        User user1 = User.builder().id(1L).name("황준오").build(); 
        User user2 = User.builder().id(2L).name("황준오").password("pass").build();
    }
}
```

&nbsp;

**빌더패턴은 계층형 클래스에서 쓰기 좋다**

- 피자 (`public abstract class Pizza`) [코드 2-4](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/hierarchicalbuilder/Pizza.java)
    + 뉴욕 피자 (`public class NyPizza extends Pizza`) [코드 2-5](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/hierarchicalbuilder/NyPizza.java)
    + 칼초네 피자 (`public class Calzone extends Pizza`) [코드 2-5](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/hierarchicalbuilder/Calzone.java)

성능상으로는 매개변수가 4개 이상일 때 값어치를 한다.

근데 빌더 생성 비용이 크진 않음.

&nbsp;
&nbsp;

### Singleton

#### 싱글톤 구현 1. static final 변수

```java
public class Util {
    public static final Util INSTANCE = new Util();

    private Util(){
    }

    //...
    // 장점 : p24
}

======================================

// 문제점 1) 리플렉션으로 객체생성 가능 (private 생성자 호출)
public class Main {
    public static void main(String[] args) {
        Constructor<?> con = Util.class.getDeclaredConstructors()[0];
        con.setAccessible(true);

        Util util1 = (Util) con.newInstance();
        Util util2 = Util.INSTANCE;

        System.out.println(util1); //Util@4554617c
        System.out.println(util2); //Util@74a14482   hashcode가 다르다!!
    }
}

======================================

// 문제점 2) 직렬화할 때 추가적인 작업 필요
// implements Serializable 만으로 직렬화를 하면
// 역직렬화할 때마다 새로운 인스턴스가 만들어짐.
```

&nbsp;

#### 싱글톤 구현 2. staic method

```java
public class Util {
    private static final Util INSTANCE = new Util();

    public static Util getInstance() {
        return INSTANCE;
    }

    //...
    // 장점 : p24
}

======================================

// 문제점 1) 리플렉션으로 객체생성 가능 (private 생성자 호출)
public class Main {
    public static void main(String[] args) {
        Constructor<?> con = Util.class.getDeclaredConstructors()[0];
        con.setAccessible(true);

        Util util1 = (Util) con.newInstance();
        Util util2 = Util.getInstance();

        System.out.println(util1); //Util@4554617c
        System.out.println(util2); //Util@74a14482   hashcode가 다르다!!
    }
}

======================================

// 문제점 2) 직렬화할 때 추가적인 작업 필요
// implements Serializable 만으로 직렬화를 하면
// 역직렬화할 때마다 새로운 인스턴스가 만들어짐.
```

&nbsp;

#### 싱글톤 구현 3. enum

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

    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}

    //...
}

======================================

public class Main {
    public static void main(String[] args) {
        Math math = new Math(); 
        // erorr! 생성자가 없음. 
        // (Math클래스의 인스턴스를 만들 방법도 없음.)
    }
}

```

애초에 `Math.class`는 유틸리티성으로 만들었다.

`Math.class` 의 필드, 메서드는 모두 **static** 이다.

&nbsp;

- 이런 유틸리티 클래스(Math, Arrays, Collections 등)에 private 생성자를 만들 것
    + 만들지 않으면 public Math() 가 생성됨
        * 실수로 객체를 생성할 여지를 줌
    + 주석도 추가해야 함
        * 생성자가 존재하는데 호출이 불가능한 것은 직관적이지 않기 때문

&nbsp;
&nbsp;


