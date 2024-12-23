# Item02: Consider a builder when faced with many constructor parameters

> [!NOTE]
> 필드가 많을 때, 생성자, 정적 팩토리 메서드로는 대응하기가 어려워진다.
> 필드가 많은 경우 유용한 Builder 에 대해 알아보겠다.

생성자 패턴으로는 크게 3 가지가 있다.

## 1. Telescoping Constructor Pattern (=점층적 생성자 패턴)

> 우선 NOT NULL 필드에 대해서만 생성자를 만들고,
> NULL 이 허용되는 필드가 하나씩 추가된 생성자를 점층적으로 만든다.

```java
public class Item {
    private final int id; // 필수
    private final String name; // 필수
    private final String category; // 필수
    private final int price; // 선택
    private final String targetCustomer; // 선택

  public Item(final int id, final String name, final String category){
      this(id, name, category, 0);
  }

  public Item(final int id, final String name, final String category, final int price){
      this(id, name, category, price, "DEFAULT");
  }

  public Item(final int id, final String name, final String category, final int price, final String targetCustomer){
      this.id = id;
      this.name = name;
      this.category = category;
      this.price = price;
      this.targetCustomer = targetCustomer;
  }
}
```

> [!WARNING]
> NULL 이 허용되는 필드가 한 개 늘어나는 경우,
> 생성자도 한 개 늘어나야 하기에 관리하기 복잡하다.

## 2. JavaBeans Pattern (= 자바빈즈 패턴)
> 각 필드에 대한 setter 를 통해 값을 세팅하는 방법이다.

```java
public class Item {
    private final int id; // 필수
    private final String name; // 필수
    private final String category; // 필수
    private final int price; // 선택
    private final String targetCustomer; // 선택
    
    public setId(final int id) {
        this.id = id;
    }
    
    public void setName(final String name) {
        this.= name;
    }
    
    public void setCategory(final String category) {
        this.category = category;
    }
    
    public void setPrice(final int price) {
        this.price = price;
    }
    
    public void setTargetCustomer(final String targetCustomer) {
        this.targetCustomer = targetCustomer;
    }
}
```
> [!warning]
> 필드 한 개마다 setter 를 호출해야 하니 메서드 호출이 많이 필요하게 된다.
> setter 를 통해 필드를 맞춰주기 전까지는 consistency 가 깨져있는 상태가 된다.
> 그렇기에 자바빈즈 패턴이 적용된 클래스는 불변으로 만들 수가 없게 된다.
> thread 환경에서 안전성을 얻기 위해서는 `freeze` 메서드를 사용해야 한다.

## 3. Builder Pattern (= 빌더 패턴)
> 필수 필드만으로 이루어진 생성자를 호출해 빌더 인스턴스를 얻는다.
> 빌더 클래스가 제공하는 setter 메서드들로 원하는 선택 필드들을 설정한다.
> 마지막으로 매개변수가 없는 build() 메서드를 호출해 필요한 인스턴스를 얻는다.

```java
public class Item {
    private final int id; // 필수
    private final String name; // 필수
    private final String category; // 필수
    private final int price; // 선택
    private final String targetCustomer; // 선택
    
    public static class Builder {
        private final int id; // 필수
        private final String name; // 필수
        private final String category; // 필수
        
        // NULL 이 허용 되는 필드는 초기값으로 초기화
        private final int price = 0; // 선택
        private final String targetCustomer = "DEFAULT"; // 선택
        
        public Builder(final int id, final String name, final String category) {
            this.id = id;
            this.name = name;
            this.category = category;
        }
        
        public Builder price(final int price) {
            this.price = price;
            return this;
        }
        
        public Builder targetCustomer(final int targetCustomer) {
            this.targetCustomer = targetCustomer;
            return this;
        }
        
        public Item build() {
            return new Item(this);
        }
    }
    
    private Item(final Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.category = builder.category;
        this.price = builder.price;
        this.targetCustomer = builder.targetCustomer;
    }
}
```

### 사용 예제
```java
Item item = new Item.Builder(1, "C", "Book")
    .price(28000)
    .targetCustomer("Student")
    .build();
```

> [!important]
> Builder 는 Telescoping Constructor Pattern 보다 간결하고, JavaBeans Pattern 보다 안전하다.
