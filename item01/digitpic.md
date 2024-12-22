# Item01: Consider static factory methods instead of constructors

## Static Factory Method
> - `public static` 으로 선언되는 `factory method` 를 의미한다.
> - `정적 팩토리 메서드` 라고 부른다.
> - 정적 팩토리 메서드를 가지는 클래스의 기본 생성자는 `private` 으로 막아둬야 한다.
>     - 그렇지 않으면, 인스턴스 생성 방식에 대한 통제를 잃게 되며 이는 클래스 설계 의도에 적합하지 않은 행위이다.

우리는 일반적으로 인스턴스를 생성하기 위해 `new` 를 이용한다.

생성자를 통해 인스턴스를 생성하는 방법도 있겠지만,
정적 팩토리 메서드를 통해 인스턴스를 생성할 수도 있다.

정적 팩토리 메서드를 사용하면 어떤 점이 좋을까?

## 장점

1. 이름을 가질 수 있다.
    - 생성자는 매개변수의 개수, 매개변수의 자료형이 동일한 생성자를 만들 수 없다.

    ```java
    /* 불가능한 경우 */
    public Book(String title){
            this.title = title;
        }

    public Book(String author){
        this.author = author;
    }
    ```

    - 정적 팩토리 메서드를 사용하면 매개변수의 개수, 매개변수의 자료형이 동일한, 서로 다른 메서드를 생성할 수 있게 된다.
    ```java
    /* 가능한 경우 */
    public static Book withAuthor(String author){
        Book book = new Book();
        book.author = author;
        return book;
    }

    public static Book withTitle(String title){
        Book book = new Book();
        book.title = title;
        return book;
    }
    ```

2. 호출될 때마다 새로운 인스턴스를 생성하지 않아도 된다.
    - 생성자는 호출될 때마다 반드시 새로운 인스턴스를 생성해야 한다.
    - 생성자처럼 인스턴스를 매번 새로 생성할 수도 있고, Singleton 패턴, Flyweigth 패턴처럼 기존 인스턴스를 재사용할 수도 있다.
    - 인스턴스 생성에 대한 비용이 높은 경우, 성능을 크게 향상시킬 수 있다.
    - 자원 절약과 유연성 확보가 가능하다.

3. return 타입의 서브 타입에 해당하는 인스턴스를 반환할 수 있다.
    - 반환 인스턴스의 클래스를 자유롭게 선택할 수 있게 되어 유연성을 확보할 수 있다.

4. 입력 매개변수에 따라 반환되는 인스턴스의 클래스가 다르게 할 수 있다.
    - 입력값에 따라 유연한 반환을 제공할 수 있다.

5. 정적 팩토리 메서드를 작성하는 시점에, 반환될 인스턴스의 클래스가 존재하지 않아도 된다.

정적 팩토리 메서드를 사용하면 나쁜 점은 없을까?

## 단점

1. `public`, `protected` 생성자 없이, 정적 팩토리 메서드만 제공하는 클래스는 상속이 불가능하기에 서브 클래스를 생성할 수 없다.
    - 하지만 상속보다는 구성을 사용하도록 유도 된다는 장점이 있다.
        - 상속은 설계에 제한을 줄 수 있으며 강한 결합을 유발할 수 있다.
    - 서브 클래스를 생성할 수 없게 되면 불변 인스턴스의 무결성을 유지할 수 있게 된다는 장점이 있다.

2. 개발자가 인스턴스를 생성하기 위한 메서드를 찾아야 하는 비용이 발생한다.
    - 클래스 문서에 정적 팩토리 메서드에 대한 정보를 명확히 적어줌으로 완화할 수 있다.
    - 잘 알려진 네이밍 패턴을 사용하면 정적 팩토리 메서드임을 암시할 수 있다.

## Naming patterns for static factory methods

| name | description | example |
|:----:|:-----------:|:-------:|
|`from`|single parameter 를 받아, 해당 클래스의 인스턴스를 반환|`Stream<Integer> stream = Stream.from(list);`|
|`of`|multiple parameters 를 받아, 해당 클래스의 인스턴스를 반환|`List<Integer> list = List.of(1, 2, 3);`|
|`valueOf`|`from`, `of` 보다 더 자세한 대안|`Integer number = Integer.valueOf("123");`|
|`instance` or<br>`getInstance`|parameters를 통해 instance 를 반환|`StackWalker luke = StackWalker.getInstance(options);`|
|`create` or<br>`newInstace`|`instacne`, `getInstacne` 와 같지만, 매번 새로운 인스턴스를 생성해 반환|`Object newArray = Array.newInstance(classObject, arrayLen);`|
|`get`+`type`|`getInstance` 와 같지만, 다른 클래스에서 해당 클래스의 인스턴스를 반환|`FileStore fs = Files.getFileStore(path);`|
|`new`+`type`|`newInstance` 와 같지만, 다른 클래스에서 해당 클래스의 인스턴스를 반환|`BufferedReader br = Files.newBufferedReader(path);`|
|`type`|`getType`, `newType` 을 단순화한 표기|`List<Complaint> litany = Collections.list(legacyLitany);`|

