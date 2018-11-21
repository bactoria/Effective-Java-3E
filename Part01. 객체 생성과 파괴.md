
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
    + 칼초네 피자 (`public class Calzone extends Pizza`) [코드 2-6](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/hierarchicalbuilder/Calzone.java)

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

#### 객체 주입 방식

유틸리티성으로 쓰이는 클래스를 설계하더라도 static으로 쓰지말아야 할 때도 있다.

```java
//코드 5-3
public class SpelChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    //...
}
```

- 객체 주입 방식
    + 위처럼 인스턴스를 생성할 때 다른 객체를 주입받는 방법.

- 코드 5-1(static), 코드 5-2(싱글턴)
    + 유연하지 않다
        * dictionary가 언어별로, 특수어휘용이 있을 수 있다.
    + 테스트하기 어렵다
        * dictionary를 대체하기 어렵다.

&nbsp;
&nbsp;

### 불필요한 객체 생성  최소화

#### 1) String

```java
// Bad! Regacy Code
public class RomanNumerals {
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
}

=============================================================

// Good !
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}

```

- Bad
    + `isRomanNumeral(String s)` 를 호출할 때마다 `새로운 String 객체`를 생성함.
- Good
    + `isRomanNumeral(String s)` 를 호출할 때마다 `ROMAN`을 참조함.

&nbsp;

#### 2) auto boxing

- JDK 1.5 이후 `AutoBoxing`과 `AutoUnBoxing`을 제공
    + AutoBoxing
        + `Long num1 = 5L;` 
        + `Long num1 = new Long(5L);`
    + AutoUnBoxing
        + `long num2 = num1;` 
        + `Long num2 = num1.intValue();`

```java
// bad!
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) 
        sum += i;

    return sum;
}

// good!
private static long sum() {
    long sum = 0L;
    for (long i = 0; i<= Integer.MAX_VALUE; i++)
        sum += i;

    return sum;
}
```

- bad
    + 15780 ms
    + for문이 한번 돌 때마다 새로운 Long 객체 생성
        * 책에는 **231**개의 불필요한 Long 객체 생성한다고 되어 있음
        * `Integer.MAX_VALUE` 개 일줄 알았는데..
        * TODO :: Long 에 대해서 모르는게 있는 것 같음.
- good
    + 3031 ms

&nbsp;
&nbsp;

### 객체 반납

사용하지 않는 객체는 GC에 의해 메모리 반납함.

반납하지 않으면 메모리 누수를 일으킨다.

&nbsp;

#### GC 대상

1. **모든 객체 참조가 null 인 경우**
2. 객체가 블럭 안에서 생성되고 블럭이 종료된 경우
3. 부모 객체가 null이 된 경우, 자식 객체는 자동적으로 GC 대상이 된다.
4. 객체가 Weak 참조만 가지고 있을 경우
5. 객체가 Soft 참조이지만 메모리 부족이 발생한 경우
출처: [IT 마이닝](http://itmining.tistory.com/24)

```java
    // 객체 생성 ( + 참조)
    List list = new ArrayList();

    // 참조 제거
    list = null;

    //이제 ArrayList는 GC대상이다.
```

**Stack.class** 에는 

```java
Stack stack<User> = new Stack<>();

stack.push(user1);
stack.push(user2);
stack.push(user3);

stack.pop(); //이 때 user3의 참조를 null처리 한다.
```

TODO :: 캐시와 리스너,콜백도 메모리 누수를 일으키는 주범.

캐시 : WeakHashMap 사용.
리스너,콜백 : weak reference로 콜백 저장.
[백기선님 유튜브](https://youtu.be/YijcBaS4cu8)

&nbsp;

#### 자원 반납

- **finalizer**
    + 자바9에서는 사용자제 API로 지정됨
    + 오동작, 낮은 성능, 이식성 문제
        + 쓰지 마시오...
- **cleaner**
    + 자바9 이후 사용가능
    + 여전히 예측할 수 없고, 느리고, 불필요

- **finalizer & cleaner**
    + 즉시 수행된다는 보장이 없음
        + GC에 따라 천차만별
    + 중요하지 않은 네이티브 자원 회수용으로만 사용할것
        + 불확실성, 성능저하 여전히 문제

&nbsp;

**try-finally**
```java
// 코드 9-1
//bad!
BufferedReader br = new BufferedReader(new FileReader(path));
try{
    return br.readLine();
} finally {
    br.close();
}
```

- try-finally가 이중 삼중으로 중첩되면 복잡함.
    + 코딩 실수할 가능성 큼.

**try-with-reference**
```java
// 코드 9-3
//good!
try (BufferedReader br = new BufferedREader(new FileReader(path))) {
    return br.readLine();
}
```

- 정확하고 쉽게 자원회수 가능
    + try () 안에 자원 넣기






