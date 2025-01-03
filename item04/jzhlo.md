## [블로그 정리 글](https://jhzlo.tistory.com/60)
## 🧐 들어가기 전...

코드를 구성하다 보면, 정적 메서드와 정적 필드의 묶음으로 클래스를 구성하고 싶을 때가 찾아온다.

```java
// 비인스턴스화 가능한 유틸리티 클래스
public class MathUtility {
    // 상수 필드
    public static final double PI = 3.14159;
    public static final double E = 2.71828;

    // 정적 메서드: 두 수의 최대값 계산
    public static int max(int a, int b) {
        return (a > b) ? a : b;
    }

    // 정적 메서드: 두 수의 최소값 계산
    public static int min(int a, int b) {
        return (a < b) ? a : b;
    }

    // 정적 메서드: 제곱 계산
    public static double square(double x) {
        return x * x;
    }

    // 정적 메서드: 팩토리얼 계산
    public static long factorial(int n) {
        if (n < 0) throw new IllegalArgumentException("n must be non-negative");
        long result = 1;
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }
}
```

위와 같은 코드처럼 수학적인 연산을 하나의 클래스에 모아둔 후,

필요한 연산만 상황에 따라 꺼내어서 쓰고자 하는 클래스를 구성해야 할 때가 있다.

우리는 이러한 클래스를 보편적으로 **Utility Class**로 분류한다.

> **Utility Class** : 정적 메서드와 정적 필드만으로 구성된 클래스

이러한 클래스는 상태를 가지고 캡슐화된 객체들의 동작으로 코드가 진행되는 것이 아닌,

전역적으로 메서드를 정의하고 절차적으로 동작하기에, 객체 지향적인 사고와는 거리가 좀 멀다.

그러나 Utility Class가 유용한 경우도 있다.

다음은 대표적인 예시이다.

-   **java.util.Math**
-   **java.util.Arrays**
-   **java.util.Collections**

그렇다면 이러한 Utility Class를 사용할 때 주의해야할 점은 무엇일까?

---

## ✅ Utility Class의 문제점

### 1️⃣ Utility Class의 특성

우선 Utility Class의 문제점을 파악하기 전에, Utility Class의 특징을 다시 한번 살펴봐야 한다.

```
1. 정적 메서드로 이루어져 있다.
2. 정적 필드로 이루어져 있다.	
3. 특정 데이터나 동작을 공통적으로 재사용하기 위해 설계된 클래스이다.
```

Utility Class는 **static**으로 이루어진 클래스이다.

다시 말해, 코드 내에 모든 객체가 동일한 정적 필드와 정적 메서드를 공유하게 된다.

따라서 **ClassName.methodName()** 또는 **ClassName.fieldName** 형태로 접근 가능하므로, **객체 생성 없이 사용**할 수 있다.

그렇기에 Utility Class에서는 **인스턴스 생성이 의미 없다.**

오히려 인스턴스 생성은 **불필요한 메모리 소비를 유발**한다.

또한, Utility Class는 상태 또한 가지지 않기에 인스턴스를 생성하려고 시도하는 것은 **클래스의 사용 목적과 어긋난다**.

---

### 2️⃣ 문제점

만약 이러한 Utility Class의 생성자를 정의하지 않게 된다면 어떤 일이 발생할까?

```java
public class UtilityClass {
    // 정적 메서드
    public static void printHello() {
        System.out.println("Hello, Utility Class!");
    }
}
```

Java에서 클래스에 **생성자가 하나도 정의되지 않은 경우**, 컴파일러는 **기본 생성자**를 자동으로 생성한다.

```java
public utilityClass() {
    super();
}
```

따라서 위와 같은 기본 생성자가 생성되므로, **생성자를 정의하지 않게 되면 인스턴스화를 막을 수 없다.**

---

### 3️⃣ 해결책 : abstract Class?

class를 추상클래스로 지정하게 되면 인스턴스를 막을 수 있을까?

```java
abstract class UtilityClass{
    // 정적 메서드
    public static void printHello() {
        System.out.println("Hello, Utility Class!");
    }
}
```

abstract 클래스는 인스턴스화가 불가능하여 표면적으로는 인스턴스화를 방지하는 것처럼 보일 수 있다.

하지만, **위와 같이 정의하는 경우에는 다음과 같은 문제점**이 있다.

---

#### ☑️ 1. abstract의 의미와 역할에 부적합

> **abstract 키워드 :**   
> 추상 메서드나 추상 클래스를 정의하기 위한 키워드로, 클래스가 설계 상 상속되어야 함을 암시한다.

단순히 인스턴스를 방지하고자 하는 목적으로 **abstract를 사용하는 것은 클래스의  설계 의도를 왜곡시키는 결과**를 초래한다.

또한 **유틸리티 클래스는 일반적으로 상속을 목적으로 하지 않는다**.

그러므로, abstract로 인스턴스를 방지하는 것은 적합하지 않다.

#### ☑️ 2. 인스턴스화 우회 가능

```
class SubUtility extends UtilityClass {
    public SubUtility() {
        System.out.println("SubUtility instance created.");
    }
}
```

두 번째 문제점은 위의 코드처럼, **UtilityClass의 상속을 받아 우회할 수 있다는 점**이다.

따라서 **abstract 키워드만으로는 완벽하게 인스턴스화를 방지할 수 없다**.

---

## ✅ private로 생성자를 선언하라.

```java
public class UtilityClass {
	// private 생성자
    private UtilityClass(){
    	throw new AssertionError();
    }
    
    public static void printHello() {
        System.out.println("Hello, Utility Class!");
    }
}
```

private로 생성자를 선언하여 인스턴스를 방지하는 방식은 위의 문제점들을 모두 해결한다.

#### ☑️ 인스턴스화 방지

**외부 및 내부에서 클래스 인스턴스화 불가능하다.**

AssertionError()를 생성자 내에서 던짐으로써, 내부에서 실수로 인스턴스를 생성하더라도

인스턴스가 생성되는 것을 방지할 수 있다.

#### ☑️ 상속이 불가능하다.

Java에서 **상속된 클래스의 생성자는 항상 부모 클래스의 생성자를 호출**해야 한다.

부모 클래스의 생성자가 private로 선언된 경우,

자식 클래스는 이를 호출할 수 없기 때문에 **상속을 통한 인스턴스화도 차단된다.**

---

#### 🚩 결론

private 생성자를 정의하면,

**직접적인 인스턴스화뿐 아니라 상속을 통한 인스턴스화 우회도 방지**할 수 있다.

이는 **유틸리티 클래스**처럼 인스턴스화가 필요 없는 클래스에 적합한 설계 방식이다.

---

## 📢 REF

이팩티브 자바 3/E
