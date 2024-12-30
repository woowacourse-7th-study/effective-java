## [블로그 정리 글](https://jhzlo.tistory.com/59)
## 🧐 들어가기 전...

#### _싱글톤 패턴이란?_

```
싱글톤: 클래스가 단 "하나의 인스턴스"만 가지도록 보장
```

하나의 인스턴스만 가졌을 때의 장점은 "일관성"에서 비롯된다.

예를 들어, 우리나라의 화폐가 발행되는 곳은 중앙은행인 한국은행에서만 발행된다.

만약에 화폐를 한국은행에서만이 아닌 여러 곳에서 찍어낸다면 일관성이 깨지게 될 것이다.

통화의 흐름을 파악하기가 어렵게 되고, 부정확한 화폐가 유통될 수도 있다.

즉, 하나의 기관에서 모든 것을 관리함으로써 일관성과 신뢰성을 챙길 수 있다.

> 싱글톤의 대상은 무상태(stateless)객체 혹은 설계상 유일해야 하는 시스템 컴포넌트가 된다.  
>  \[이팩티브 자바 中\]

이처럼, 코드를 짤 때에도 **하나의 인스턴스만 가지도록 보장해야하는 경우가 있따른다**.

이는 여러 인스턴스를 허용하지 않음으로써 **충돌과 혼란**을 방지하고, 시스템 전체에 통일성을 제공한다.

---

## ✅ Singleton을 만드는 방법

### 1️⃣  public static final

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
    
    public static void main(String[] args){
    	Elvis elvis = Elvis.INSTANCE;
    }
}
```

첫 번째로 소개하는 방식은 위와 같이 필드값을 public으로 지정하고 static으로 Instance를 생성한 후에,

직접적으로 해당 인스턴스에 접근하는 방법이다.

> 이때 생성자는 private로 막아 필드값에서 정의한 인스턴스에만 접근할 수 있도록 설정한다.

---

### 2️⃣ 정적 팩토리 메서드

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() { ... }
    
    public static Elvis getInstance(){
        return INSTANCE;
    }
    
    public static void main(String[] args){
    	Elvis elvis = Elvis.getInstance();
    }
}
```

두 번쨰의 방식은 **static factory method**를 이용하는 방식이다.

구현하는 방식은 다음과 같다.

1.  필드 값에 **private static**으로 **Instance**를 생성한다.
2.  **private**로 기본 생성자를 막아둔다.
3.  **getInstance()** 메서드를 **public static**으로 두고, 필드에 생성해둔 **Instance**를 return한다.

---

#### 🚩 리플렉션(Reflection)으로 싱글톤 깨뜨리기

**그러나** 위와 같은 방식은 리플렉션(Reflection)으로 인한 인스턴스가 생성될 수 있다는 단점이 있다.

```
리플렉션: 리플렉션은 Java의 강력한 기능으로, 런타임 동작의 유연성과 동적 객체 조작을 가능하게 한다.
(Java에서 리플렉션은 주로 java.lang.reflect 패키지를 통해 제공된다.)
```

리플렉션의 기능은 다음과 같다.

1.  **클래스 정보 탐색**
2.  **생성자 탐색 및 호출**
3.  **필드 접근 및 수정**
4.  **메서드 호출**
5.  **애노테이션 탐색**

따라서 해당 리플렉션의 기능, 생성자를 탐색하고 필드를 수정하게 된다면 다음과 같이 싱글톤을 깨뜨릴 수 있게 된다.

> public static final 싱글톤 class 깨뜨리기

```java
import java.lang.reflect.Constructor;

public class ReflectionTest {
    public static void main(String[] args) throws Exception {
        // 기존 싱글톤 인스턴스 가져오기
        Elvis instance1 = Elvis.INSTANCE;

        // 리플렉션으로 private 생성자 접근
        Constructor<Elvis> constructor = Elvis.class.getDeclaredConstructor();
        constructor.setAccessible(true); // private 접근 가능하게 설정

        // 새로운 인스턴스 생성
        Elvis instance2 = constructor.newInstance();

        // 인스턴스 비교
        System.out.println("instance1 == instance2: " + (instance1 == instance2)); // false
    }
}
```

> 정적 팩토리 메서드 싱글톤 class 깨뜨리기

```java
// 싱글톤 클래스
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// 리플렉션으로 싱글톤 파괴
Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true); // private 생성자 접근 허용
Singleton instance1 = Singleton.getInstance();
Singleton instance2 = constructor.newInstance();

System.out.println(instance1 == instance2); // false
```

---

#### 🚩 직렬화(Serialization)로 싱글톤 깨뜨리기

```java
import java.io.Serializable;

public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}

    // 역직렬화 시 항상 기존 인스턴스를 반환
    private Object readResolve() {
        return INSTANCE;
    }
}
```

만약 위의 코드처럼 직렬화로 싱글톤 클래스가 구현된 경우에는 **역직렬화**를 통해 싱글톤을 꺠뜨릴 수 있다.

```java
import java.io.*;

public class SerializationTest {
    public static void main(String[] args) throws Exception {
        // 기존 싱글톤 인스턴스
        Elvis instance1 = Elvis.INSTANCE;

        // 직렬화
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("elvis.obj"));
        out.writeObject(instance1);
        out.close();

        // 역직렬화
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("elvis.obj"));
        Elvis instance2 = (Elvis) in.readObject();
        in.close();

        // 인스턴스 비교
        System.out.println("instance1 == instance2: " + (instance1 == instance2)); // false
    }
}
```

직렬화된 객체를 역직렬화하면 새로운 인스턴스가 생성되므로 싱글톤 보장이 깨진다.

---

## ✅ Singleton 보장하기

### 1️⃣ Reflection 방어하기

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private static boolean instanceCreated = false;

    private Elvis() {
        if (instanceCreated) {
            throw new IllegalStateException("이미 인스턴스가 존재합니다!");
        }
        instanceCreated = true;
    }
}
```

**생성자 내부에서 중복 생성 방지 로직**을 추가하여 reflection으로 싱글톤을 깨뜨리는 방식을 방어할 수 있다.

---

### 2️⃣ 직렬화 방어하기

```java
import java.io.Serializable;

public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}

    // 역직렬화 시 항상 기존 인스턴스를 반환
    private Object readResolve() {
        return INSTANCE;
    }
}
```

**readResolve()** 메서드를 활용하여 역직렬화 시 새로운 객체가 아니라 기존에 만들어놓은 싱글톤 객체를 반환하도록 설정하여 방어할 수 있다.

---

### 3️⃣ enum으로 방어하기

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("Elvis가 건물을 떠났습니다!");
    }
}
```

enum으로 싱글톤 클래스를 구현하면 다음과 같은 이점을 지닌다.

-   INSTANCE는 enum 요소로 선언되며, Java는 enum 요소를 **JVM 차원에서 단 한 번만 생성**하도록 보장한다.


-   기본적으로 **public static final** 속성을 가지므로 별도의 싱글톤 구현이 필요하지 않다.


-   생성자가 자동으로 **private**으로 설정되므로 외부에서 객체 생성이 불가능gkek.

따라서 enum을 사용하여 싱글톤 클래스를 구현하게 된다면,

리플렉션과 직렬화를 통해 새로운 객체를 생성할 수 없다.

#### ☑️ 장점

Enum 싱글톤 클래스의 장점을 정리해보자면 다음과 같다.

> **📌JVM 보장**  
> enum은 JVM 수준에서 단일 인스턴스 생성을 보장하며, 리플렉션이나 직렬화 공격으로부터 안전하다.
>
> (1) 리플렉션 공격 방어  
> enum의 생성자는 리플렉션으로 접근할 수 없도록 Java 언어가 막고 있다.
>
> (2) 직렬화 공격 방어  
> enum은 별도의 readResolve() 메서드를 구현하지 않아도 JVM이 자동으로 동일한 인스턴스를 반환한다.
>
> **📌 구현 간결성**  
> 복잡한 방어 로직 없이도 단순한 코드로 싱글톤을 구현할 수 있다.
>
> **📌Thread Safe**  
> 멀티스레드 환경에서도 동기화 문제를 신경 쓸 필요가 없다.
>

#### ☑️ 단점

반면 enum의 싱글톤 클래스에 대해 다음과 같은 단점을 가지고 있기도 하다.

> **📌클래스 확장이 불가능**  
> enum은 다른 클래스를 상속할 수 없다.
>
> **📌초기화 논리 제한**  
> enum은 간단한 싱글톤 구현에 적합하지만, 복잡한 초기화 로직이 필요한 경우 설계가 복잡해질 수 있다.

---

#### 📢 REF

이팩티브 자바 3/E
