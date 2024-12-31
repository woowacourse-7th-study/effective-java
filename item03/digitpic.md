# Item 3: Enforce the singleton property with a private constructor or an enum type
> [!note]
> singleton 적용할 때,
> 생성자를 private 으로 막거나 enum type singleton 을 사용할 것

## singleton
- 인스턴스를 하나만 생성할 수 있는 클래스
- singleton 은 mock 객체를 활용한 테스트 어려움
- mock 객체를 사용하려면 singleton 이 interface 를 구현해야 함

## singleton 구현 방법 3가지
### 1. `public final` 필드를 사용하는 방법
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {
        
    }
    
    public void leaveTheBuilding() {
        
    }
}
```

- 클래스의 `public static final` 필드로 싱글톤 인스턴스 제공
- 코드가 간결함
- API를 통해 싱글톤임이 명확
- 리플렉션을 통한 무단 생성 공격 가능
    - 이를 방지하려면 생성자에서 두번째 인스턴스 생성 시 예외를 던지게 해야 함
    ```java
    public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
        
    private Elvis() { 
          if( INSTANCE != null) {
            throw new RuntimeException("Can't create Constructor");
            }    
        }
    }    
    ```
    
### 2. `정적 팩토리 메서드`를 사용하는 방법
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {
        
    }
    
    public static Elvis getInstance() {
        return INSTANCE; 
    }
    
    public void leaveTheBuilding() {
        
    }
}
```

- 싱글톤 인스턴스를 반환하는 `public static` 메서드
- 싱글톤 여부를 나중에 변경할 수 있는 유연성 제공
- 메서드 참조 사용 가능
    
### 3. `Enum`을 사용하는 방법 (권장)
```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() {
        
    }
}
```

- 싱글톤 인스턴스를 열거형의 요소로 선언
- 코드가 간결함
- 직렬화와 리플렉션 공격에 강함
- 상속이 필요한 경우는 사용할 수 없음

---

싱글톤 클래스에서 직렬화를 지원하려면
모든 필드를 transient로 선언하고
readResolve 메서드를 추가해야 함

그렇지 않으면 직렬화-역직렬화 과정에서
새로운 인스턴스가 생성될 수 있음

