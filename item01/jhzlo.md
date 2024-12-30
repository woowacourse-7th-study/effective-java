### [블로그 정리](https://jhzlo.tistory.com/56)

> [!NOTE]
> 
> 이 책에서 성능에 집중하는 부분은 많지 않다.
> 
> 대신 프로그램을 명확하고, 정확하고, 유용하고, 견고하고, 유연하고 관리하기 쉽게 짜는 데 집중한다.

# Item01. 생성자 대신 static 팩토리 메소드를 고려해 볼 것
## ✅ Static Factory Method(정적 메소드)

## 🧐들어가기전..
java에서 객체를 생성하기 위한 방법으로는 크게 두 가지가 있다.
1. `생성자 (constructor)`
```java
class Car{
    private final String name;
    
    // 생성자
    public car(String name){
        this.name = name;
    }
}
```
2. `정적 팩토리 메서드`
```java
class Car{
    private final Car INSTANCE = new Car();
    
    private Car(){
    }
    
    // 정적 팩토리 메서드
    public static Car getInstance(){
        return INSTANCE;
    }
}
```

두 가지의 방식이 있다는 것은 각각의 장단점이 있다는 것이고,
우리는 상황에 맞게 적합한 방식을 선택할 줄 알아야 한다.

**그렇다면 정적 팩토리의 메서드의 장단점에 대해 한 번 알아보자.**

---

## ☑️ 장점
### 1️⃣ 이름을 가질 수 있다.
### ❗ 가독성의 향상
```java
class Car{
    private final String name;

    // 생성자
    public car(String name){
        this.name = name;
    }
}
```
생성자는 클래스 이름과 동일한 이름을 가진다.

즉, 생성자의 매개변수 String이 Car의 어떤 상태값에 매핑이 되는지 알 수 없다.

```java
class Car{
    private final String name;
    
    private Car(String name){
        this.name = name;
    }
    
    // 정적 팩토리 메서드
    public static Car withName(String name){
        return new Car(name);
    }
}
```
하지만 위의 코드처럼 정적 팩토리 메서드를 사용하게 되는 경우에는,
`withName`과 같이 객체 생성에 의미를 부여할 수 있다.

이는 개발하는 사람들의 입장에서 매개변수의 값이 Car의 어떤 상태 값에 해당하는지에 대한 가독성을 향상시켜준다.

그리고 **가독성 향상은 유지보수성과도 직결**된다.

### ❗ 시그니처 구분
```java
class Car{
    private String name;
    private String color;
    
    private car(String name){
        this.name = name;
    }
    
    // 불가능
    private car(String color){
        this.color = color;
    }
}
```
생성자는 같은 시그니처 타입을 가지고 생성하는 것이 불가능하다.

```java
class Car{
    private String name;
    private String color;
    
    private car(){
    }
    
    public static Car withName(String name){
        Car car = new Car();
        car.name = name;
        return car;
    }

    public static Car withColor(String color){
        Car car = new Car();
        car.color = color;
        return car;
    }
}
```
하지만, 정적 팩토리 메서드를 활용한다면 같은 시그니처를 가지더라도 

**이름을 가질 수 있기에** 각각의 상황에 맞게 서로 다른 생성자를 만들어낼 수 있다는 장점을 가지고 있다.

_즉, 생성자보다 더 유연한 성질을 지니고 있다._
### 2️⃣ 반드시 새로운 객체를 만들 필요가 없다.

```java
class CarFactory {
    private boolean isWork = true;
    public static final CarFactory INSTANCE = new CarFactory();
    
    private carFactory(){
    }
    
    public static CarFactory getInstance(){
        return INSTANCE;
    }
}
```
"반드시 새로운 객체를 만들 필요가 없다는 것"이 어떠한 점에서 이점이 있을까?

불변(immutable) 클래스인 경우나 매번 새로운 객체를 만들 필요가 없는 경우에 미리 만들어둔 인스턴스 또는 캐시해둔 인스턴스를 반환할 수 있다

```text
즉 객체를 하나로만 관리하면 관리의 편리함과 일관성을 얻을 수 있다. 
(ref. Singleton Pattern)
```

예를 들어, 위와 같은 코드에서 `CarFactory`는 자동차를 생성하는 공장을 나타내는 객체이다.

CarFactory는 자동차 공장의 작동 여부를 나타내는 `isWork`라는 상태값을 가지고 있다.

만약 이 공장이 여러 객체로 생성된다면, 다음과 같은 문제가 발생할 수 있습니다.

1. `상태의 불일치`
```text
여러 CarFactory 객체가 서로 다른 상태값을 가지게 되어, 
어느 공장이 자동차를 생산 중인지 명확하지 않게 된다.
```

2. `관리의 복잡성`
```text
상태값을 여러 곳에서 수정할 수 있으므로, 
자동차 공장을 관리해야 하는 CarFactory의 역할이 모호해지고 관리가 어려워진다.
```

결론적으로, `Singleton Pattern`은 관리가 간편하고 일관성이 중요한 객체를 설계할 때 매우 유용하다. 

CarFactory와 같은 객체를 `정적 팩토리 메서드`로 관리하면, `"오직 하나의"` 객체를 통해 모든 상태와 동작을 통제할 수 있어 **설계의 명확성과 안정성**을 높일 수 있다.
또한 더 나아가 **자원을 효율적으로 사용**할 수 있다.

### 3️⃣ 리턴 타입의 하위 타입 인스턴스를 만들 수도 있다.
```java
public interface MyInterface {
    void doSomething();
}

class MyImplementation implements MyInterface {
    public void doSomething() {
        System.out.println("Doing something!");
    }
}

public class Main {
    public static void main(String[] args) {
        MyInterface instance = new MyInterface();
        instance.doSomething(); // "Doing something!" 출력
    }
}
```
어찌보면 결국 위의 내용들의 연장선이다.

`MyInterface`의 상속을 받은 `MyImplementation`을 main 함수에서 실행시키려면 위의 방식이 보편적이다.

하지만 이렇게 되는 경우 **MyInterface의 하위 타입 인스턴스가 외부로 노출**된다는 단점이 있다.

따라서 이러한 노출을 감추고 하위 instance의 메서드를 호출하고 싶을 떄에는

```java
public interface MyInterface {
    void doSomething();
}

class MyImplementation implements MyInterface {
    public void doSomething() {
        System.out.println("Doing something!");
    }
}

class MyFactory {
    public static MyInterface createInstance() {
        return new MyImplementation(); // 인터페이스의 구현체를 반환
    }
}

public class Main {
    public static void main(String[] args) {
        MyInterface instance = MyFactory.createInstance();
        instance.doSomething(); // "Doing something!" 출력
    }
}
```
`MyFactory` 클래스에서 정적 팩토리 메서드를 선언함으로써 인터페이스 구현체를 반환해주고,

이를 main함수에서 정적 팩토리 메서드를 이용하여 하위 인스턴스를 불러오는 방법대로 진행한다면,
**구현을 숨김으로써 API 설계의 복잡성을 줄이는 효과를 가져온다.**

### ❗자바 8 vs 자바 9
1. `java 8`
```java
public interface MyInterface {
    static MyInterface create() {
        return new MyImplementation();
    }
}
```
java 8부터는 위와 같이 인터페이스 내에서 구현체를 반환하는 정적 메서드를 제공할 수 있다.
2. `java 9`

인터페이스에 private static 메서드를 추가할 수 있다.

이를 통해 정적 메서드에서만 사용하는 유틸리티 로직을 캡슐화할 수 있다.


### ❗ java.util.Collections 사례
java.util.Collections는 45개의 인터페이스 구현체를 제공하지만, 
이 구현체들은 모두 **non-public**이다. 
즉, 클라이언트는 구현체에 대해 알 필요가 없고, 인터페이스를 통해서만 기능을 사용할 수 있다.

```java
List<String> list = Collections.unmodifiableList(new ArrayList<>(Arrays.asList("A", "B", "C")));
```
위 코드에서 `unmodifiableList`는 내부적으로 Collections.UnmodifiableList라는 non-public 구현체를 반환한다. 
그러나 클라이언트는 List 인터페이스만 보게 된다.

이렇게 함으로써 다음과 같은 이점을 챙길 수 있다.

1. `API 단순화`

    클라이언트는 인터페이스만 알면 되므로 `"개념적인 무게"`가 줄어든다.

    (여기서 개념적인 무게란, 프로그래머가 어떤 인터페이스가 제공하는 API를 사용할 때 알아야 할 개념의 개수와 난이도를 말한다.)


2. `구현체 변경의 유연성`

    내부 구현체를 클라이언트와 독립적으로 변경할 수 있어 유지보수가 용이하다.

    예를 들어, 성능을 개선하기 위해 구현체를 바꿔도 클라이언트 코드에 영향을 주지 않는다.


### 4️⃣ 리턴하는 객체의 클래스가 입력 매개변수에 따라 매번 다를 수 있다.
```java
public interface Car {
    void drive();
}

class RegularCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving a regular car.");
    }
}

class LuxuryCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving a luxury car with premium features.");
    }
}
```
입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다.

(예를 들어, 특정 조건에 따라 간단한 Car 객체를 반환하거나, 더 복잡한 기능을 가진 LuxuryCar 객체를 반환할 수 있다.)

위와 같은 코드에서 정적 팩토리 메서드를 활용하여 다음과 같이 **매개변수에 따른 다른 객체를 생성할 수 있다**.
```java
public class CarFactory {
    public static Car createCar(int budget) {
        if (budget < 50_000) {
            return new RegularCar(); // 예산이 낮을 경우 RegularCar 반환
        } else {
            return new LuxuryCar(); // 예산이 높을 경우 LuxuryCar 반환
        }
    }
}
```

### ❗JDK 사례
```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```
`EnumSet`의 경우, 내부적으로 열거형의 개수에 따라 `RegularEnumSet` 또는 `JumboEnumSet`을 반환한다.

이렇게 함으로써 조건에 따라 객체를 더 유연하게 생성할 수 있다.
### 5️⃣ 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
정적 팩토리 메서드가 리턴하는 객체의 클래스가 팩토리 메서드를 작성할 당시 반드시 존재하지 않아도 된다는 `유연성`에 대한 내용이다.

이 유연성은 `서비스 프로바이더 프레임워크`의 핵심 요소로, 
실제 객체의 구현체를 나중에 등록하거나 동적으로 로드할 수 있게 해준다.

1. `서비스 인터페이스`
```java
public interface Car {
    void drive();
}
```
서비스 인터페이스는 특정 작업을 수행할 구현체를 규정한다.

2. `서비스 프로바이더 인터페이스`
```java
public interface CarProvider {
    Car createCar();
}
```
실제 구현체를 제공하는 인터페이스다. (자동차 생성)

3. `서비스 프로바이더 등록 API`
```java
public class CarFactory {
    private static final Map<String, CarProvider> providers = new HashMap<>();

    // 프로바이더 등록 API
    public static void registerProvider(String name, CarProvider provider) {
        providers.put(name, provider);
    }

    // 서비스 액세스 API
    public static Car getCar(String name) {
        CarProvider provider = providers.get(name);
        if (provider == null) {
            throw new IllegalArgumentException("No provider registered with name: " + name);
        }
        return provider.createCar();
    }
}
```
구현체를 등록하는 API 이다.

등록된 프로바이더가 클라이언트 요청에 따라 자동차 객체를 생성할 수 있다.

4. `구현체: 다양한 자동차 클래스`
```java
public class SportsCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving a fast sports car!");
    }
}

public class ElectricCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving an eco-friendly electric car!");
    }
}
```
실제 자동차의 구체적인 구현체는 나중에 작성되거나 동적으로 등록된다.

5. `클라이언트 코드: 프로바이더 등록 및 사용`
```java
public class Main {
    public static void main(String[] args) {
        // 서비스 프로바이더 등록
        CarFactory.registerProvider("sports", SportsCar::new);
        CarFactory.registerProvider("electric", ElectricCar::new);

        // 서비스 액세스 API를 통해 자동차 생성
        Car sportsCar = CarFactory.getCar("sports");
        sportsCar.drive();

        Car electricCar = CarFactory.getCar("electric");
        electricCar.drive();
    }
}
```

위의 코드에 대한 예시
```text
1. 서비스 인터페이스
자동차를 타는 사람이 자동차의 "인터페이스"를 정의한다. 

예를 들어:
자동차는 가속해야 한다.
자동차는 브레이크를 밟아야 한다.
자동차는 방향을 바꿀 수 있어야 한다.
여기서 중요한 것은 구체적으로 어떤 자동차인지 정의하지 않는다는 점이다.

2. 서비스 프로바이더 인터페이스
자동차를 공급하는 렌터카 회사이다.
렌터카 회사는 손님에게 자동차를 제공하지만, 손님은 자동차의 브랜드나 세부 사양에 대해 알 필요가 없다.
손님이 "SUV를 원해요"라고 말하면 렌터카 회사가 적합한 SUV를 제공한다.

3.프로바이더 등록 API
렌터카 회사는 다양한 자동차 회사(제조사)와 계약한다.
"현대자동차 SUV", "테슬라 전기차", "페라리 스포츠카" 등 여러 자동차를 등록할 수 있다.
특정 요구사항에 따라 적합한 자동차를 공급하도록 준비한다.

4.서비스 액세스 API
손님이 렌터카 카운터에 와서 "예산이 30만 원입니다. SUV를 원합니다"라고 요청한다.
렌터카 회사는 요구에 맞는 자동차를 꺼내온다. 

예를 들어:
예산이 적으면 "현대 싼타페" 제공.
예산이 높으면 "BMW X5" 제공.
```
즉 위와 같이 구현함으로써 다음과 같은 이점을 챙길 수 있다.
1. **구현체가 팩토리 작성 시점에 존재하지 않아도 된다.**
   - `CarFactory`를 설계할 때 SportsCar나 ElectricCar와 같은 구현체가 없더라도, 이후에 구현하고 등록하면 된다.
   - 새로운 구현체를 추가하려면 단순히 새로운 클래스를 작성하고 `registerProvider`로 등록하면 된다.
2. **동적 확장 기능**
3. **서비스 인터페이스와 구현체 분리**

### ❗JDBC 사례

| **제공부**                         | **서비스부**                       |
|---------------------------------|-----------------------------------|
| **서비스 제공자 인터페이스**               | **서비스 인터페이스**             |
| *(service provider interface)*  | *(service interface)*             |
| `Driver`                        | `Connection`                      |
|                                 |                                   |
| **제공자 등록 API**                  | **서비스 접근 API**               |
| *(provider registration API)*   | *(service access API)*            |
| `DriverManager.registerDriver()` | `DriverManager.getConnection()`   |

JDBC의 경우, DriverManager.registerDriver()가 프로바이더 등록 API. 

DriverManager.getConnection()이 서비스 엑세스 API. 

그리고 Driver가 서비스 프로바이더 인터페이스 역할을 한다.

제공자 등록 API 역할을 하는 DriverManager.registerDriver()메서드로 등록한 제공자(Driver)에 맞는 Connection 서비스를 반환한다.

즉 mySql Driver, Oracle Driver 등 DB에 따라 다른 Connection을 제공한다.

---

## ☑️ 단점
### 1️⃣ public 또는 protected 생성자 없이 static public 메소드만 제공하는 클래스는 상속할 수 없다.
public 또는 protected 생성자가 없는 클래스는 상속할 수 없다.

예시:
- java.util.Collections의 정적 메서드(unmodifiableList, emptyList 등)는 편리하지만, 해당 클래스는 상속할 수 없다.
- 상속이 필요한 상황에서는 사용할 수 없다는 제약이 있다.

_하지만 이는 역으로 장점이 될 수도 있다..._

### 2️⃣ 프로그래머가 static 팩토리 메소드를 찾는게 어렵다.
정적 팩토리 메서드가 잘 노출되지 않으므로, 문서화 및 설명을 명확히 추가해야 한다.

---

## 🔍 주로 사용하는 명명 방식
| 메서드      | 설명                                                                                   | 예제                                                                                       |
|-------------|---------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| `from`      | 매개변수를 하나 받아 해당 타입의 인스턴스를 반환(형변환 method)                         | `Date d = Date.from(instant);`                                                            |
| `of`        | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드                      | `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`                                    |
| `valueOf`   | `from`과 `of`의 더 자세한 버전                                                        | `BigInteger.valueOf(Integer.MAX_VALUE);`                                                  |
| `instance` / `getInstance` | 매개변수를 받을 경우 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스를 보장하지는 않음 | `StackWalker luke = StackWalker.getInstance(options);`                                    |
| `create` / `newInstance` | `instance` 혹은 `getInstance`와 같지만 매번 새로운 인스턴스를 생성해 반환함   | `Object newArr = Array.newInstance(classObj, arrayLen);`                                  |
| `getType`   | `getInstance`와 같으나 생성할 클래스가 아닌 다른 클래스의 팩터리 메서드를 정의할 때 사용 | `FileStore fs = Files.getFileStore(path);`                                                |
| `newType`   | `newInstance`와 같으나 생성할 클래스가 아닌 다른 클래스의 팩터리 메서드를 정의할 때 사용 | `BufferedReader br = Files.newBufferedReader(path);`                                      |
| `type`      | `getType`와 `newType`의 간결한 버전                                                   | `List<Complaint> litany = Collections.list(legacyLitany);`                                |
