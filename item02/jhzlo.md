### [블로그 정리](https://jhzlo.tistory.com/57)

# Item02. 생성자 매개변수가 많은 경우에 빌더 사용을 고려해 볼 것

## 🧐들어가기전..
> [!NOTE]
> 
> 클래스가 가지고 있는 필드 변수가 많을 때, 
> 
> 선택적인 필드만 초기화하는 생성자를 만들려면 어떻게 해야할까?

```java
public class Character {
    private final String name;
    private final int faceSize;
    private final int neckSize;
    private final int bodySize;
    private final int armSize;
    private final int legSize;

    public Character(String name) {
        this(name, 0);
    }
    
    ...
}
```
만약 정적팩토리를 사용한다면 선택적 매개변수가 많다면 사용하기가 힘들 것이다.

따라서 이러한 문제를 해결하고자 세 가지 패턴에 대해 알아볼 것이다.

---
## 1️⃣ 점층적 생성자 패턴(telescoping constructor pattern)
```java
public class NutritionFacts {

	private final int servingSize; // (ml, 1회 제공량) 필수
	private final int servings; // (회, 총 n회 제공량) 필수
	private final int calories; // (1회 제공량당) 선택
	private final int fat; // (g/1회 제공량) 선택
	private final int sodium; // (mg/1회 제공량) 선택
	private final int carbohydrate; // (g/1회 제공량) 선택
```

위와 같이 필수 매개변수와 선택 매개변수가 있다고 가정하자.

이때 점층적 생성자 패턴이란.
- 필수 매개변수만 받는 생성자
- 필수 매개변수 + 선택 매개변수 1개
- 필수 매개변수 + 선택 매개변수 2개
- ...
```java
public class NutritionFacts {

    private final int servingSize; // (ml, 1회 제공량) 필수
    private final int servings; // (회, 총 n회 제공량) 필수
    private final int calories; // (1회 제공량당) 선택
    private final int fat; // (g/1회 제공량) 선택
    private final int sodium; // (mg/1회 제공량) 선택
    private final int carbohydrate; // (g/1회 제공량) 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
위와 같은 방식대로 매개변수를 점점 늘려가며`this()`를 이용한 `점층적 생성자 패턴, 생성자 체이닝`을 이용한다.

### ☑️ 단점
- **매개변수가 늘어날수록 생성자가 많아진다.**
- **`가독성 저하`**
  - 보통 0 같은 기본값으로 다른 매개변수의 값을 채운다.
  - 몇 번째 인자가 어떤 값을 의미하는지 알기 어렵다.
- **`같은 시그니처의 생성자`는 만들지 못하므로 완벽한 제어를 할 수 없다.**

## 2️⃣ 자바빈즈 패턴(JavaBeans pattern)

### 🚩 자바빈이란?

```text
자바빈 : 자바빈(JavaBean)은 자바에서 재사용 가능한 소프트웨어 컴포넌트를 만들기 위한 표준을 따르는 클래스이다.
```
특징
- `기본 생성자 제공`
  - JavaBean 클래스는 기본 생성자(파라미터가 없는 생성자)를 반드시 포함해야함
- `프로퍼티 접근자 메서드`
  - Getter
  - Setter
- `직렬화 가능`
  - java.io.Serializable 인터페이스를 구현
- `캡슐화`
  - 모든 프로퍼티는 private로 선언

###  📢 자바빈즈 패턴?
```text
자바빈즈 패턴: 매개 변수가 없는 생성자 (기본 생성자)로 객체를 만든 후 setter 메서드를 호출하여 원하는 매개변수의 값을 설정하는 방식이다.
```
```java
public class NutritionFacts {

	// 매개변수들은 기본값이 있다면 기본값으로 초기화된다.
	private int servingSize = -1; // 필수, 기본값 없음
	private int servings = -1; // 필수, 기본값 없음
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	// 생성자
	public NutritionFacts() {
	}

	// setter
	public void setServingSize(int servingSize) {
		this.servingSize = servingSize;
	}

	public void setServings(int servings) {
		this.servings = servings;
	}

	public void setCalories(int calories) {
		this.calories = calories;
	}

	public void setFat(int fat) {
		this.fat = fat;
	}

	public void setSodium(int sodium) {
		this.sodium = sodium;
	}

	public void setCarbohydrate(int carbohydrate) {
		this.carbohydrate = carbohydrate;
	}
    
}
```
위와 같이 기본생성자를 하나 두고 나머지의 프로퍼티에 대해서는 Setter로 지정할 수 있게 구현하는 방식이다.

### ☑️ 장점
1. **인스턴스 생성이 간단해진다.**
   - 기본 생성자만으로도 객체를 생성할 수 있다.
2. **필요한 매개변수만 세팅할 수 있다.**
   - 생성자 체이닝과 다르게 선택적으로 매개변수를 세팅할 수 있다.
3. **가독성이 좋다.**

### ☑️ 단점
1. **필수적인 매개변수를 지정할 수 없다.**
   - 필수로 설정해줘야 하는 필드가 초기화되지 않고 객체가 생성될 여지가 높다.
2. **완전한 객체를 만들기 위해서는 여러 개의 setter 메서드를 호출해야한다.**
   - 어떤 한 setter 메서드를 빼먹어서 객체가 불완전한 상태로 사용될 여지가 있다.
3. **클래스를 `불변`으로 만들 수 없다.**
   - setter 메서드의 치명적인 단점은 외부에서 클래스의 프로퍼티를 임의로 바꿀 수 있다는 점이다.
   - `객체 프리징 기술`로 보완가능하지만 현업에서는 잘 안씀
```java
    public void setLegSize(int legSize) {
        validateSetter();
        this.legSize = legSize;
    }
           
    private void validateSetter() {
        if (freeze) {
            throw new IllegalStateException("더이상 필드를 설정할 수 없습니다.");
        }
    }
           
    public void freeze() {
        freeze = true;
    }
```
위와 같은 방식으로 freeze를 한 번 호출하면 setter 메서드 사용이 불가능하다.

## 3️⃣ 빌더 패턴(Builder pattern)
>[!NOTE]
> 
>점층적 생성자 패턴과 자바빈즈 생성자 패턴의 단점을 보완할 수 있는 방식

빌더패턴의 원리는 다음과 같습니다.
- 생성자(혹은 정적 팩토리) 메서드를 호출해 필수 매개변수만으로 빌드 객체를 얻습니다.
- 빌더 객체가 제공하는 일종의 `Setter 메서드`들로 원하는 선택 매개변수를 선택한다.
- 마지막으로 매개변수가 없는 `build 메서드`를 호출해 필요한 `불변의 객체`를 얻는 방식이다.

```java
public class NutritionFacts {

	// 빌더 패턴(Builder pattern)
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수 - 기본값으로 초기화
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) {
			calories = val;
			return this;
		}

		public Builder fat(int val) {
			fat = val;
			return this;
		}

		public Builder sodium(int val) {
			sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}

}
```
>[!NOTE]
> 빌더 클래스는 보통 생성할 클래스 안에 정적 멤버 클래스로 만든다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
	.calories(100).sodium(35).carbohydrate(27).build();
```
위와 같이 `Builder`의 `Setter 메서드`들은 `this() 연산자`를 통한 자신을 반환하기 때문에 연쇄적으로 사용이 가능하다.

(=플루언트 API(fluent API) 혹은 메서드 연쇄(method chaining))

### ☑️ 장점
1. **점층적 생성자 패턴보다 클라이언트 입장에서 훨씬 이해하기 쉽다.**
   - 생성자의 매개 변수를 통해 필수적인 프로퍼티가 무엇인지 파악 가능
```java
public class NutritionFacts {

	// 빌더 패턴(Builder pattern)
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;
```
2. **자바빈즈 패턴보다 안정적이다.**
   - 메서드 체이닝 끝에 생성되는 객체는 모든 필드가 final인 ‘불변 객체’ 로 반환받을 수 있다.
3. **매개변수의 값들을 유기적으로 설정해줄 수 있다.**
   - 가변인수 변수를 여러 개 사용할 수 있다.
     - Builder 클래스에서 Setter 메서드를 여러 개로 확장시킴에 따라 가능하다.
   - 메서드 여러 개로부터 받은 값들을 한 필드로 모을 수 있다.
     - 메서드 체이닝을 통해 매개변수의 값을 하나씩 해당 필드에 추가할 수 있다.

### ☑️ 단점
1. 성능에 민감한 상황에서 문제가 될 수 있다.
   - 코드만 봐도 훨씬 길어졌으니..
   - 매개변수가 많을수록 위의 단점을 상쇄시킬 수 있다..
2. 구현하기가 어렵다.
   - 이에 대한 해결책으로는 `@builder` 롬복 어노테이션을 사용할 수 있다.
```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class NutritionFacts {

    // 필드
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

}
```
Builder 어노테이션을 적용하면 위와 같이 코드가 간결해진다.

그러나 Builder 패턴에도 치명적인 단점이 있다.
컴파일된 코드를 살펴보면 다음과 같다.
```java
import java.util.Objects;

public class NutritionFacts {

    // 필드
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    // Getter 메서드 (롬복의 @Getter에 의해 생성됨)
    public int getServingSize() {
        return servingSize;
    }

    public int getServings() {
        return servings;
    }

    public int getCalories() {
        return calories;
    }

    public int getFat() {
        return fat;
    }

    public int getSodium() {
        return sodium;
    }

    public int getCarbohydrate() {
        return carbohydrate;
    }

    // @Builder가 생성한 내부 정적 Builder 클래스
    public static class NutritionFactsBuilder {
        private int servingSize;
        private int servings;
        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;

        // Builder 메서드
        public NutritionFactsBuilder servingSize(int servingSize) {
            this.servingSize = servingSize;
            return this;
        }

        public NutritionFactsBuilder servings(int servings) {
            this.servings = servings;
            return this;
        }

        public NutritionFactsBuilder calories(int calories) {
            this.calories = calories;
            return this;
        }

        public NutritionFactsBuilder fat(int fat) {
            this.fat = fat;
            return this;
        }

        public NutritionFactsBuilder sodium(int sodium) {
            this.sodium = sodium;
            return this;
        }

        public NutritionFactsBuilder carbohydrate(int carbohydrate) {
            this.carbohydrate = carbohydrate;
            return this;
        }

        // build() 메서드
        public NutritionFacts build() {
            return new NutritionFacts(this.servingSize, this.servings, this.calories, this.fat, this.sodium, this.carbohydrate);
        }

        @Override
        public String toString() {
            return "NutritionFacts.NutritionFactsBuilder(servingSize=" + this.servingSize + ", servings=" + this.servings
                    + ", calories=" + this.calories + ", fat=" + this.fat + ", sodium=" + this.sodium + ", carbohydrate="
                    + this.carbohydrate + ")";
        }
    }

    // NutritionFacts의 생성자
    private NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }

    // Builder 생성 메서드
    public static NutritionFactsBuilder builder() {
        return new NutritionFactsBuilder();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof NutritionFacts)) return false;
        NutritionFacts that = (NutritionFacts) o;
        return servingSize == that.servingSize && servings == that.servings && calories == that.calories &&
                fat == that.fat && sodium == that.sodium && carbohydrate == that.carbohydrate;
    }

    @Override
    public int hashCode() {
        return Objects.hash(servingSize, servings, calories, fat, sodium, carbohydrate);
    }

    @Override
    public String toString() {
        return "NutritionFacts(servingSize=" + this.servingSize + ", servings=" + this.servings + ", calories=" + this.calories
                + ", fat=" + this.fat + ", sodium=" + this.sodium + ", carbohydrate=" + this.carbohydrate + ")";
    }
}

```
1. `@Builder`에서는 필수 매개변수를 강제하지 않는다.
    - 결국 @Builder와 별도의 검증 로직을 추가하거나, 필수 매개변수를 초기화하는 정적 메서드를 작성해야 한다.
2. 성능 오버헤드
3. 디버깅이 어려움

> [!IMPORTANT]
> 클래스의 필드값이 4개 이상인 경우에는 생성자나 정팩메 대신 빌더패턴을 고려해보자.
