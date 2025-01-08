## [블로그 정리 글](https://jhzlo.tistory.com/61)

## 🧐 들어가기 전...

> 많은 클래스는 하나 이상의 기본 자원에 의존한다.

**객체 지향 언어에서는 각각의 객체들이 서로를 필요로 하며, 비즈니스 로직을 수행해나간다.**

객체는 서로의 메서드를 호출하거나 데이터를 교환하면서 협력하며,

이러한 **상호작용을 통해 애플리케이션의 주요 기능을 구현**한다.

이 과정에서 하나의 객체가 다른 객체의 인스턴스를 생성하고 의존하는 구조는 자연스럽게 나타나지만,

**잘못된 설계 방식으로 인해 코드의 유연성과 재사용성이 저하될 위험**이 있다.

**단순히 new 생성자를 통해서 인스턴스를 생성하는 것은 잘못된 방식**일까?

그렇다면 **어떤 방식으로 객체의 인스턴스를 생성해야하고, 어떻게 의존성을 관리해야 올바른 설계를 할 수 있을까**?

---

## ✅ 기존의 문제점 

### 1️⃣ 정적 유틸리티 클래스의 부적절한 사용

```
public class SpellChecker {
	private static final Lexicon dictionary = new Lexicon();
	
	// 인스턴스화 불가
	private SpellChecker() {
		throw new AssertionError();
	}
	
	public static boolean isValid(String word) {
		return false;
	}
	
	public static List<String> suggestions(String type){
		return null;
	}
}
```

**정적 유틸리티 클래스**:

클래스의 메서드를 모두 static으로 만들어 한 번 정의된 자원(예: 사전)을 바꿀 수 없게 된다.

즉, 항상 고정된 자원을 사용하게 되므로, **다른 언어의 사전**이나 **테스트용 사전** 같은 다양한 자원을 사용할 수 없다.

---

### 2️⃣ 싱글톤의 부적절한 사용

```
public class SpellChecker {
	private final Lexicon dictionary = new Lexicon();
	public static SpellChecker2 INSTANCE = new SpellChecker2();
	
	// 인스턴스화 불가
	private SpellChecker2() {
		throw new AssertionError();
	}
	
	public boolean isValid(String word) {
		return false;
	}
	
	public List<String> suggestions(String type){
		return null;
	}
	
	public static void main(String[] args) {
		SpellChecker2 spellChecker2 = SpellChecker2.INSTANCE;
		spellChecker2.isValid("test");
	}
}
```

**싱글톤 패턴**:

자원(예: 사전)을 하나의 고정된 객체로 만들어 클래스 전체에서 공유하도록 강제한다.

이렇게 하면 여러 개의 자원을 사용하는 **다양한 인스턴스**를 생성할 수 없게 된다.

---

위에서 다루는 두 방식이 적절하지 않은 이유는 다음과 같다.

-   **유연성 부족**  
    정적 유틸리티 클래스와 싱글톤 클래스에서 사용하는 사전 Lexicon이 하나의 인스턴스로 제한된다.  
    하지만, 사전 특성상 여러 군데에서 손 쉽게 접근 가능해야하므로 위와 같은 방식은 유연성을 떨어뜨린다.
-   **테스트 어려움**  
    언어별, 용도별로 다른 자원이 필요하거나 테스트 시 특정 자원을 사용하는 경우에 적합하지 않다.
-   **확장성 저하**  
    동작을 매개변수화하기 어렵고, 동시성 환경에서 오류가 발생하기 쉽다.

> 🚩**NOTE**  
> 클래스의 동작이 특정 자원(예: 사전, 데이터베이스, 설정 파일 등)에 따라 달라질 경우,  
> 정적 유틸리티 클래스나 싱글톤 패턴을 사용하면 유연성이 부족해지고 문제가 생긴다.

---

## ✅ 의존성 주입을 사용하라

의존성 관리가 복잡해질수록 객체 간 결합도가 높아지고 테스트가 어려워질 수 있다.

이를 해결하기 위해 등장한 것이 **의존성 주입(Dependency Injection)** 이다.

의존성 주입은 객체가 직접 필요한 인스턴스를 생성하는 대신, 외부에서 주입받도록 설계하는 방식으로,

객체 간의 결합도를 낮추고 설계의 유연성과 테스트 가능성을 크게 향상시킨다.

### 1️⃣ 의존성 주입 구현

```
public class SpellChecker {
	private final Lexicon dictionary;

	// 의존성 주입
	public SpellChecker3(Lexicon dictionary) {
		this.dictionary = dictionary;
	}
	
	public boolean isValid(String word) {
		System.out.println("call is Valid");
		return false;
	}
	
	public List<String> suggestions(String type){
		System.out.println("call suggestions");
		return null;
	}
}
```

의존성 주입은 위와 같이 생성자에 객체를 "주입"함으로써 객체간의 의존성을 생기게 한다.

이는 외부에서 객체를 주입했다고 표현하는데 왜 그럴까?

```
public class Main {
	public static void main(String[] args) {
    	Lexicon dictionary = new Lexicon();
        SpellChecker spellChecker = new SpellChecker(dictionary); // 외부에서 주입
        
        spellChecker.isValid("hello");
    }
}
```

외부에서 객체를 주입했다고 표현하는 이유는,

객체가 자신이 직접 의존성을 생성하거나 관리하지 않고 외부의 다른 주체가 해당 객체를 만들어서 제공하기 때문이다.

이는 **객체가 자신의 책임을 명확히 하고** 의존성 생성과 같은 부가적인 작업에서 벗어나 **자신의 역할에만 집중하게 한다.**

> 🚩 **외부에서 객체를 주입받는 설계는 책임의 분리를 통해 코드의 유지보수성과 확장성을 높이는 데 기여한다.**

---

### 2️⃣ 자원 팩토리를 활용한 의존성 주입

```
public class SpellChecker {
    private final Lexicon dictionary;

    // 의존성 주입 (자원 팩토리 활용)
    public SpellChecker(Supplier<? extends Lexicon> dictionaryFactory) {
        this.dictionary = Objects.requireNonNull(dictionaryFactory.get());
    }

    public boolean isValid(String word) {
        System.out.println("call isValid");
        return false;
    }

    public List<String> suggestions(String type) {
        System.out.println("call suggestions");
        return null;
    }
}
```

의존성 주입의 변형된 패턴 중 하나는 자원 팩토리(Resource Factory)를 생성자에 전달하는 방식이다.

> **자원 팩토리**: 특정 타입의 객체를 반복적으로 생성하거나 제공할 수 있는 역할을 담당한다.

해당 코드에서의 특이한 점은 **Supplier<T>** 의 사용이다.

Supplier<T>는 **매개변수를 받지 않고 값을 반환하는** 기능을 제공한다.

이 인터페이스는 주로 객체를 **지연 생성**하거나 **동적으로 제공**할 때 사용된다.

```
Supplier<Car> carFactory = Car::new;
Car car = carFactory.get(); // 새로운 Car 객체 생성
```

따라서 위와 같은 코드처럼 쉽게 팩토리 메서드 패턴을 구현할 수 있다.

이러한 특징은 **객체 생성 로직을 캡슐화하면 코드 중복을 줄이고, 유지보수성을 높일 수 있다.**

---

### 3️⃣ 대규모 프로젝트에서의 의존성 주입

의존성 주입은 객체 간의 결합도를 낮추고 유연성과 테스트 가능성을 크게 향상시키는 설계 방식이다.

하지만, **대규모 프로젝트에서는 이로 인해 새로운 도전 과제와 복잡성이 발생할 수 있다**.

#### 🚩 대규모 프로젝트에서의 문제점

대규모 애플리케이션은 수천 개의 클래스와 객체들로 구성되며, 이들 간의 의존 관계는 매우 복잡할 수 있다.

이러한 상황에서 **수동으로 의존성을 관리**하는 방식은 다음과 같은 문제를 초래할 수 있다

1.  **코드의 가독성 저하**
    -   의존성을 직접 생성자나 팩토리로 관리하다 보면 코드가 지저분해지고 관리가 어려워진다.
2.  **유지보수의 어려움**
    -   새로운 의존성을 추가하거나 기존 의존성을 변경할 때, 관련된 코드 전반을 수정해야 할 수 있다.

---

#### 🧐 해결 방안 : 의존성 주입 프레임워크

의존성 주입 프레임워크는 객체 간 의존성을 자동으로 설정하고 관리하는 도구이다.

이러한 프레임워크를 사용하면 **복잡한 의존 관계를 단순화하고, 유지보수성과 확장성을 높일 수 있다**.

**대표적인 의존성 주입 프레임워크**

1.  **Dagger**
    -   구글에서 개발한 경량 DI 프레임워크.
    -   정적 컴파일 타임에 의존성을 확인하고 생성하므로 런타임 오버헤드가 적음.
    -   Android 개발에 자주 사용.
2.  **Guice**
    -   구글이 개발한 Java 기반 DI 프레임워크.
    -   런타임에 의존성을 주입하며, 간단한 설정과 강력한 확장성을 제공.
    -   Java 서버 애플리케이션에서 널리 사용.
3.  **Spring**
    -   가장 널리 사용되는 DI 프레임워크로, Java의 대규모 프로젝트에서 표준으로 자리 잡음.
    -   DI뿐만 아니라 웹 애플리케이션, 데이터베이스, 메시징 등 다양한 기능을 제공.

---

## 📢 REF

이팩티브 자바 3/E
