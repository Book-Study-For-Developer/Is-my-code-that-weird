## 1. 반드시 현재 클래스의 인스턴스 변수 사용하기

인스턴스 변수를 안전하게 조작하도록 메서드를 설계하면, 클래스 내부가 정상적인 상태인지 보장할 수 있다.

[다른 클래스의 인스턴수 변수를 변경하는 메서드는 좋지 않다.](https://www.notion.so/5-7534e25de9a14a77b65ab37cb45d169d?pvs=21) 다른 클래스의 인스턴스 변수를 변경하는 메서드를 작성할 때는 변경된 내용을 다루는 새로운 인스턴스를 생성하고 이를 리턴하는 것이 좋다.(불변성과 관련있는 듯)

## 2. 불변을 활용해서 예상할 수 있는 메서드 만들기

가변 인스턴스 변수 등을 변경하는 메서드는 의도하지 않게 다른 부분에 영향을 줄 수 있다.

**불변을 활용해서 예상치 못한 동작 자체를 막을 수 있게 설계하자**

## 3. 묻지 말고 명령하라

**[equipArmor 메서드](https://www.notion.so/5-7534e25de9a14a77b65ab37cb45d169d?pvs=21)**처럼 ‘다른 클래스를 확인하고 조작하는 메서드 구조’는 응집도가 낮은 구조이다.

```tsx
class Person {
    private name: string;

    getName(): string {
        return name;
    }

    setName(newName: string): void {
        this.name = newName;
    }
}
```

**getter/setter는 ‘다른 클래스를 확인하고 조작하는 메서드 구조’가 되기 쉽다.**

[Equipment 클래스](https://www.notion.so/5-7534e25de9a14a77b65ab37cb45d169d?pvs=21)처럼 호출되는 메서드 쪽에서 처리를 하는 형태로 만드는 것이다.

### 4. 커멘드/쿼리 분리

이 메소드는 상태 변경과 추출을 동시에 진행하고 있다.

```tsx
gainAndGetPoint(): number {
	this.point += 10;
	return point;
}
```

메서드는 커맨드 또는 쿼리 중에 하나만 하도록 설계해야 한다는 패턴이 **커멘드/쿼리 분리(CQS, Command-Query Seperation)**이다.

| 메서드 종류 구분 | 설명                           |
| ---------------- | ------------------------------ |
| 커맨드           | 상태를 변경하는 것             |
| 쿼리             | 상태를 리턴하는 것             |
| 모디파이어       | 커멘드와 쿼리를 동시에 하는 것 |

커맨드와 쿼리로 분리해 코드를 단순화 시키자

```tsx
gainPoint(): void {
	this.point += 10;
}

getPoint(): number {
	return point;
}
```

## 5. 매개변수

-   불변 매개변수로 만들기
    -   매개변수에 final 수식자를 붙여서 불면으로 만들어라
    -   JS에서는 매개변수를 불변으로 하는 방법은 따로 없다.
-   [플래그 매개변수 사용하지 않기](https://www.notion.so/6-13a8c0a01927497c9e87f776f00e71b4?pvs=21)
    -   플래그 매개변수를 받는 메서드는 코드를 읽는 사람이 메서드가 무슨일을 하는지 알기 어렵게 만든다.
-   [null 전달하지 않기](https://www.notion.so/9-37b2ec2f2d9247ea85cdb8780005ac59?pvs=21)
    -   null을 확인해야 하므로 로직이 복잡해지는 문제를 발생시킨다.
-   [출력 매개변수 사용하지 않기](https://www.notion.so/5-7534e25de9a14a77b65ab37cb45d169d?pvs=21)
    -   출력 매개 변수를 사용하면 응집도가 낮은 구조가 만들어진다.
-   매개변수는 최대한 적게 사용하기
    -   메서드가 많다는 것은 여러 가지 기능을 처리한다는 의미’

## 6. 리턴 값

### ‘자료형’을 사용해서 리턴 값의 의도 나타내기

```tsx
class Price {
    add(other: Price): number {
        return this.amount + other.amount;
    }
}
```

단순한 기본 자료형으로는 리턴값의 의미를 호출하는 쪽에 전달할 수 없다.

```tsx
let price = productPrice.add(otherPrice);
let discountedPrice = calcDiscountedPrice(price);
let deliveryPrice = calcDeliveryPrice(discountedPrice);
```

모든 자료형이 `number` 면 매개변수를 잘못 전달하는 등의 실수가 발생할 수 있다.

```tsx
// DeliveryCharge에는 배송비가 전달되어야 하는데
// 상품 가격 합계가 전달디고 있다.

const deliveryChage = new DeliveryCharge(price);
```

add메서드의 리턴형을 Price 자료형으로 변경해 의도를 명확히 한다.

```tsx
class Price {
    add(other: Price): Price {
        const added = this.amount + other.amount;
        return new Price(added);
    }
}
```

### null 리턴하지 않기

매개변수로 null을 전달하지 않는 것이 좋듯이 ,null을 리턴하지 않아야 좋다.

### 오류는 리턴 값으로 리턴하지 말고 예외 발생시키기

```tsx
// Bad

class Location {
    shift(shiftX: number, shiftY: number) {
        let nextX = x + shiftX;
        let nextY = y + shiftY;
        if (this.valid(nextX, nextY)) {
            return new Location(nextX, nextY);
        }
        return new Location(-1, -1);
    }
}

// Good

class Location {
    constructor(x: number, y: number) {
        if (!this.valid(x, y)) {
            throw new Error("잘못된 위치입니다.");
        }

        this.x = x;
        this.y = y;
    }

    shift(shiftX: number, shiftY: number) {
        let nextX = x + shiftX;
        let nextY = y + shiftY;

        return new Location(nextX, nextY);
    }
}
```

‘오류가 있을 때, 오류 값으로 Location(-1, -1)을 리턴한다’ 라는 사실을 알고 있어야 하며, 해당 값을 후속 로직에서 정상 값처럼 사용되어 버그가 생길 것이다. **코드가 중의적으로 해석할 만한 것은 피해야 한다.**

리턴 값으로 오류 값을 리턴하지 말고 바로 예외를 발생 시키자
