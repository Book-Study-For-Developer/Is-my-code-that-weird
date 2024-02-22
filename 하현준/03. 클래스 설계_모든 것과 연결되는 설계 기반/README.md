# 3장 클래스 설계: 모든 것과 연결되는 설계 기반

### 1. 클래스 단위로 잘 동작하도록 설계하기

클래스 하나로도 동작에 무리가 없어야 하며, 복잡한 초기 설정을 하지 않아도 되며, 최소한의 조작 방법으로 외부에 제공되어야 한다.

**클래스의 구성 요소**

-   클래스는 `인스턴스 변수` 와 `메서드` 로 구성된다.
-   잘 만들어진 클래스는 `인스턴스 변수`, `인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드` 로 구성되어 있다.

**모든 클래스가 갖추어야 하는 자기 방어 임무**

다른 클래스를 사용해 초기화와 유효성 검사를 해야 하는 클래스는 안전하게 사용할 수 없는 **미성숙한 클래스**이다.

즉, 데이터 클래스에 자기 방어 임무를 부여해서, 다른 클래스에 맡기던 일을 스스로 방어할 수 있도로 설계하면 된다.

### 2. 성숙한 클래스로 성장시키는 설계 기법

금액을 나타내는 `Money` 클래스를 성숙한 클래스로 하나씩 바꾸는 방법이다.

```tsx
class Money {
    amount: number;
    currency: Currency;
}
```

**생성자로 확실하게 정상적인 값 설정하기**

‘로우 데이터 객체(raw data object)’로서 ‘초기화되지 않은 상태’ 를 유발하는 클래스 구조이다.

적절한 초기화 로직을 생성자에 구현한다.

```tsx
class Money {
    amount: number;
    currency: Currency;

    constructor(amount: number, currency: Currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

`amount` 로 유효하지 않은 값(음수, 정수가 아닌 값) 이 올 수 있기 때문에 내부에 유효성 검사를 하도록 추가 구현한다. 즉, 가드를 통해 안전하고 정상적인 인스턴스만 존재하도록 한다.

```tsx
class Money {
    amount: number;
    currency: Currency;

    constructor(amount: number, currency: Currency) {
        if (amount < 0) throw new Error("금액은 0 이상의 값을 지정해주세요.");
        if (currency === null) throw new Error("통화 단위를 지정해 주세요.");
        this.amount = amount;
        this.currency = currency;
    }
}
```

**계산 로직도 데이터를 가진 쪽에 구현하기**

‘데이터’와 데이터를 조작하는 로직’ 이 분리되어 있는 구조를 **‘응집도가 낮은 구조’** 라고 불린다.

계산 로직을 같은 클래스내에 구현해서 성숙한 클래스로 구현한다.

```tsx
class Money {
    amount: number;
    currency: Currency;

    constructor(amount: number, currency: Currency) {
        if (amount < 0) throw new Error("금액은 0 이상의 값을 지정해주세요.");
        if (currency === null) throw new Error("통화 단위를 지정해 주세요.");
        this.amount = amount;
        this.currency = currency;
    }

    private add(other: number) {
        this.amount += other;
    }
}
```

**불변 변수로 만들어서 예상하지 못한 동작 막기**

인스턴스 변수를 계속해서 바꾼다면 예상치 못한 부수 효과가 쉽게 발생한다.

이를 막기 위해 인스턴스 변수를 불변(immutable)로 만들어야 한다.

_(책에서는 자바코드로 되어있기 때문에 `final` 키워드를 사용하지만 JS에는 없기 때문에 typescript의 `private` 과 `readonly`키워드를 사용해 외부에서 조작이 불가능하도록 만든다.)_

```tsx
class Money {
    private readonly amount: number;
    private readonly currency: Currency;

    constructor(amount: number, currency: Currency) {
        if (amount < 0) throw new Error("금액은 0 이상의 값을 지정해주세요.");
        if (currency === null) throw new Error("통화 단위를 지정해 주세요.");
        this.amount = amount;
        this.currency = currency;
    }
}
```

**변경하고 싶다면 새로운 인스턴스 만들기**

변경이 필요하다면 변경된 값을 가지고 새로운 인스턴스를 만들도록 하면 된다.

```tsx
class Money {
    //...

    private add(other: number) {
        let added = this.amount + other;
        return new Money(added, this.currency);
    }
}
```

**메서드 매개변수와 지역 변수도 불변으로 만들기**

`added` 또한 불변하는 값으로 변경해준다.

```tsx
class Money {
    //...
    private add(other: number) {
        const added = this.amount + other;
        return new Money(added, this.currency);
    }
}
```

**엉뚱한 값을 전달하지 않도록 하기**

`Money` 타입의 값이 전달되어야 하는데 엉뚱한 값이 전달될 수 있기에 올바른 자료형만 받도록 변경해야 한다.

```java
class Money {
  //...
  Money add(final Money other) {
	  final int added = amount + other.amount;
    return new Money(added, currency)
	}
}
```

```tsx
class Money {
    //...
    private add(other: Money) {
        const added = this.amount + other.amount;
        return new Money(added, this.currency);
    }
}
```

기본 자료형이 아닌 독자적인 자료형을 사용해 안전한 메서드를 만들 수 있다.

결과적으로 응집도가 높은 구조를 가진 클래스르 만들었고, 필요한 메소드만 외부에 공개하는 것을 **캡슐화**라고 한다.

### 3. 프로그램 구조의 문제 해결에 도움을 주는 디자인 패턴

**완전 생성자(complete constructor)**

-   완전 생성자는 잘못된 상태로부터 클래스를 보호하기 위한 디자인 패턴이다.
-   인스턴스 변수를 모두 초기화해야만 객체를 생성할 수 있게, 매게변수를 가진 생성자를 만든다.
-   생성자 내부에서는 가드를 사용해 잘못된 값이 들어오지 않게 만든다.

JS에서는 인스턴스를 생성하면 `constructor` 가 실행되기 때문에 2번째는 자연스럽게 해결이 된다.

**값 객체(value object)**

-   값 객체는 값을 클래스로 나타내는 디자인 패턴이다.
-   금액, 날짜, 전화번호등 다양한 값을 객체로 만들어 각각의 값과 로직을 응집도가 높은 구조로 만들 수 있다.
