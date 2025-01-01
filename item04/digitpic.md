### Item 4: Enforce noninstantiability with a private constructor

> 생성자를 private 으로 막아, 인스턴스화를 차단하는 방법에 대해 다루는 내용이다.

정적 메서드와 정적 필드만으로 구서된 클래스를 작성하고 싶은 경우가 있다.

이 클래스를 유틸리티 클래스라고 한다.

유틸리티 클래스는 객체 지향 설계를 회피하기 위한 수단으로 남용되며

좋지 않은 평판을 얻기도 했지만 적절하게 사용하면 유용하다.

관련 메서드를 그룹화하는 클래스를 유틸 클래스로 사용하면 된다.

---

이 유틸리티 클래스는 인스턴스화 되지 않도록 설계해야 한다.

클래스의 생성자를 private 으로 막아두어 인스턴스 생성을 막아야 한다.

명시적으로 생성자를 만들지 않으면 public 접근 제어자를 가진 기본 생성자가 제공되기에

인스턴스화를 방지하기 위한 경우, private 생성자를 명시적으로 추가해줘야 한다.

---

클래스의 인스턴스화를 방지하기 위해 abstract class 로 선언하는 시도도 있는데, 이는 잘못된 방법이다.

클래스를 서브클래싱하면 인스턴스를 생성할 수 있기 때문이다.

그렇기 때문에 사용자가 클래스를 상속 목적으로 설계했는지 오해하는 경우가 발생할 수 있다.

```
// 부모 클래스 (abstract)
public abstract class AbstractUtilityClass {
    // 인스턴스화를 막기 위한 시도
    protected AbstractUtilityClass() {
        System.out.println("AbstractUtilityClass 생성자 호출");
    }
}

// 자식 클래스
public class SubUtilityClass extends AbstractUtilityClass {
    // 자식 클래스의 생성자
    public SubUtilityClass() {
        super(); // 부모 클래스의 생성자를 호출
    }
}
```

```
public class Main {
    public static void main(String[] args) {
        // 자식 클래스의 인스턴스 생성
        SubUtilityClass subInstance = new SubUtilityClass();
    }
}
```

---

생성자가 private 이라도 해당 클래스 내부에서는 인스턴스를 생성할 수 있다.

이를 방지하기 위해서는 생성자 내에 AssertionError 를 호출하면 된다.

이는 클래스 내부에서 인스턴스를 생성하는 행위를 방지할 수 있게 해준다.

```
public class UtilityClass {
    // 인스턴스화를 방지하기 위해 기본 생성자를 private 으로 생성
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

---

private 생성자는 클래스의 상속도 차단하게 된다.

자식 클래스는 부모 클래스의 생성자를 호출해야 하는데,

자식 클래스가 부모 클래스의 생성자에 접근하지 못하게 되기 때문이다.

