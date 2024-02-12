## 1. 리팩터링의 흐름

리팩터링이란 실질적인 동작은 유지하면서, 구조만 정리하는 작업이다.

```tsx
class PurchasePointPayment {
    readonly custormerId: CurstomerId;
    readonly comicId: ComicId;
    readonly consumptionPoint: ConsumptionPoint;
    readonly paymentDateTime: LocalDateTime;

    constructor(customer: Customer, comic: Comic) {
        if (this.customer.isEnabled()) {
            this.customerId = customer.id;
            if (comic.isEnabled()) {
                this.comic = comic.id;
                if (
                    comic.currentPurchasePoint.amount <=
                    customer.possessionPoint.amount
                ) {
                    this.consumptionPoint = comic.currentPurchasePoint;
                    this.paymentDateTime = new Date();
                } else {
                    throw new Error("보유하고 있는 포인트가 부족합니다.");
                }
            } else {
                throw new Error("현재 구매할 수 없는 만화입니다.");
            }
        } else {
            throw new Error("유효하지 않은 계정입니다.");
        }
    }
}
```

### 중첩을 제거하여 보기 좋게 만들기

PurchasePointPayment 클래스에서 조건 판정을 위해 if 조건문을 여러 번 중첩하고 있다.
이러한 중첩문을 없애보자.

```tsx
class PurchasePointPayment {
    readonly custormerId: CurstomerId;
    readonly comicId: ComicId;
    readonly consumptionPoint: ConsumptionPoint;
    readonly paymentDateTime: LocalDateTime;

    constructor(customer: Customer, comic: Comic) {
        if (!this.customer.isEnabled()) {
            throw new Error("유효하지 않은 계정입니다.");
        }

        this.customerId = customer.id;
        if (!comic.isEnabled()) {
            throw new Error("현재 구매할 수 없는 만화입니다.");
        }

        this.comic = comic.id;
        if (
            comic.currentPurchasePoint.amount > customer.possessionPoint.amount
        ) {
            throw new Error("보유하고 있는 포인트가 부족합니다.");
        }

        this.consumptionPoint = comic.currentPurchasePoint;
        this.paymentDateTime = new Date();
    }
}
```

### 의미 단위로 로직 정리하기

서로 다른 일이 뒤섞여 있으므로, 로직이 정리되지 않는다. **조건 확인**과 **값 대입 로직**을 분리하여 정리한다.

```tsx
class PurchasePointPayment {
    readonly custormerId: CurstomerId;
    readonly comicId: ComicId;
    readonly consumptionPoint: ConsumptionPoint;
    readonly paymentDateTime: LocalDateTime;

    constructor(customer: Customer, comic: Comic) {
        if (!this.customer.isEnabled()) {
            throw new Error("유효하지 않은 계정입니다.");
        }

        if (!comic.isEnabled()) {
            throw new Error("현재 구매할 수 없는 만화입니다.");
        }

        if (
            comic.possessionPoint.amount < customer.currentPurchasePoint.amount
        ) {
            throw new Error("보유하고 있는 포인트가 부족합니다.");
        }

        this.customerId = customer.id;
        this.comic = comic.id;
        this.consumptionPoint = comic.currentPurchasePoint;
        this.paymentDateTime = new Date();
    }
}
```

### 조건을 읽기 쉽게 하기

유효하지 않은 구매자 계정에 논리 부정 연산자 ‘!’를 사용하고 있어 한 번 더 생각하게 만든다.

isDisabled하는 메서드를 만들어 호출하도록 수정한다.

```tsx

	constructor(customer: Customer, comic: Comic) {
		if (this.customer.isDisabled()) {
			throw new Error("유효하지 않은 계정입니다.");
		}

		if (comic.isDisabled()) {
			throw new Error("현재 구매할 수 없는 만화입니다.");
		}

		if (comic.possessionPoint.amount < customer.currentPurchasePoint.amount) {
			throw new Error("보유하고 있는 포인트가 부족합니다.");
		}

		this.customerId = customer.id;
		this.comic = comic.id;
		this.consumptionPoint = comic.currentPurchasePoint;
		this.paymentDateTime = new Date();
	}
```

### 무턱대고 작성한 로직을 목적을 나타내는 메서드로 바꾸기

목적을 나타내는 메서드(`isShortOfPoint`)로 만들어 사용한다.

```tsx
class Customer {
    readonly id: CustomerId;
    readonly possessionPoint: PurchasePoint;

    isShortOfPoint(comic: Comic): boolean {
        return possessionPoint.amount < comicCurrentPurchasePoint.amount;
    }
}
```

```tsx
class PurchasePointPayment {
    readonly custormerId: CurstomerId;
    readonly comicId: ComicId;
    readonly consumptionPoint: ConsumptionPoint;
    readonly paymentDateTime: LocalDateTime;

    constructor(customer: Customer, comic: Comic) {
        if (!this.customer.isEnabled()) {
            throw new Error("유효하지 않은 계정입니다.");
        }

        if (!comic.isEnabled()) {
            throw new Error("현재 구매할 수 없는 만화입니다.");
        }

        if (customer.isShortOfPoint(comic)) {
            throw new Error("보유하고 있는 포인트가 부족합니다.");
        }

        this.customerId = customer.id;
        this.comic = comic.id;
        this.consumptionPoint = comic.currentPurchasePoint;
        this.paymentDateTime = new Date();
    }
}
```

## 2. 단위 테스트로 리팩터링 중 실수 방지하기

단위 테스트는 작은 기능 단위로 동작을 검증하는 테스트를 의미한다.

테스트 코드가 없는 프로덕션 코드에서 테스트 코드를 작성하고 리팩터링 하는 과정으로 설명한다.

다음은 온라인 쇼핑몰에서 배송비를 계산하고, 리턴하는 메서드이다.

```tsx
class DeliveryManager {
    static deliveryCharge(products: Array<Product>) {
        let charge = 0;
        let totalPrice = 0;
        for (each of Procuts) {
            totalPrice += each.price;
        }

        if (totalPrice < 20000) {
            charge = 5000;
        } else charge = 0;
        return charge;
    }
}
```

### 코드 과제 정리하기

이 메서드는 static으로 정의되어 있고, static 메서드는 데이터와 데이터를 조작하는 로직을 분리해서 정의할 수 있는 구조여서 응집도가 낮아지기 쉽다.

-   ‘배송비’는 금액을 나타내는 개념이므로, 값 객체로 만들자
-   합계 금액은 장바구니를 확인할 때, 실제로 주문할 때 등 다양한 유스케이스에 사용된다.
    -   이를 별도의 클래스로 빼자
-   합계 금액 계산은 일급 컬렉션 패턴으로 설계

### 테스트 코드를 사용한 리팩터링 흐름

1. 이상적인 구조의 클래스 기본 형태를 어느 정도 잡는다.
2. 이 기본 형태를 기반으로 테스트 코드를 작성한다.
3. 테스트를 실패시킨다.
4. 테스트를 성공시키기 위한 최소한의 코드를 작성한다.
5. 기본 형태의 클래스 내부에서 리팩터링 대상 코드를 호출한다.
6. 테스트가 성공할 수 있도록, 조금씩 로직을 이상적인 구조로 리펙터링한다.

**이상적인 구조의 클래스 기본 형태를 어느정도 잡는다.**

장바구니 클래스

```tsx
class ShoppingCart {
    readonly products: Array<Product>;

    constructor(products?: Array<Product>) {
        if (products) {
            this.products = products;
        } else this.products = new Array();
    }

    add(prouct: Product) {
        const adding = new Array();
        adding.push(product);

        return new ShoppingCart(adding);
    }
}
```

상품 클래스

```tsx
class Product {
    readonly id: number;
    readonly name: string;
    readonly price: number;

    constructor(id: number, name: string, price: number) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
}
```

배송비 클래스

```tsx
class DeliveryCharge {
    readonly amount: number;

    constructor(shoppingCart: ShoppingCar) {
        this.amount = -1;
    }
}
```

**테스트코드 작성하기**

배송비 책정 기준 테스트코드 작성하기

-   상품 합계 금액이 20,000원 미만이면, 배송비는 5,000원입니다.
-   상품 합계 금액이 20,000원 이상이면, 배송비는 무료입니다.

```tsx
describe("배송비 책정 기준을 테스트합니다.", () => {
    test("상품 합계 금액이 20,000원 미만이면, 배송비는 5,000원입니다.", () => {
        const emptyCart = new ShoppingCart();
        const oneProductAdded = emptyCart.add(new Product(1, "상품A", 5000));
        const twoProductAdded = oneProductAdded.add(
            new Product(2, "상품B", 14990)
        );

        const charge = new DeliveryCharge(twoProductAdded);

        expect(charge.amount).toBe(5000);
    });

    test("상품 합계 금액이 20,000원 이상이면, 배송비는 무료입니다.", () => {
        const emptyCart = new ShoppingCart();
        const oneProductAdded = emptyCart.add(new Product(1, "상품A", 5000));
        const twoProductAdded = oneProductAdded.add(
            new Product(2, "상품B", 15000)
        );

        const charge = new DeliveryCharge(twoProductAdded);

        expect(charge.amount).toBe(0);
    });
});
```

**테스트 실패시키기**

단위 테스트는 프로덕션 코드를 구현하기 전에, 실패와 성공을 확인해야 한다.
지금 현재의 코드로는 테스트는 둘 다 실패할 것이다.

**테스트** **성공시키기**

테스트를 성공시키기 위한 최소한의 코드만 구현한다. 그러기 위해서는 `DeliveryCharge` 클래스를 변경해야 한다.

```tsx
class DeliveryCharge {
    readonly amount: number;

    constructor(shoppingCart: ShoppingCart) {
        const totalPrice =
            shoppingCart.products.get(0).price + shoppingCart.products(1).price;
        if (totalPrice < 20000) amount = 5000;
        else amount = 0;
    }
}
```

설계는 부실하더라도 일단은 이렇게 작성환다.

**리팩터링하기**

테스트가 성공했으니 리팩터링을 진행한다.

```tsx
class DeliveryCharge {
    readonly amount: number;

    constructor(shoppingCart: ShoppingCart) {
        this.amount = DeliveryManager.deliveryCharge(shopingCart.products);
    }
}
```

totalPrice 메서드를 ShoppingCart 클래스에 추가한다.

```tsx
class ShoppingCart {
    // 생략

    totalPrice(): number {
        let amount = 0;
        for (each of products) {
            amount += each.price;
        }
        return price;
    }
}
```

`DeliveryManager`에서 shoppingCart를 그대로 받아 배송비를 측정하도록 구현한다.

```tsx
class DeliveryManager {
	static deliveryCharge(shoppingCart: ShoppingCart) {
		let charge = 0;
		if (shoppingCart.totalPrice() < 20000) {
			charge = 5000;
		} else charge = 0;

		return charge;
}

// shoppingCart를 그대로 받으니 같이 변경한다.
class DeliveryCharge {
	readonly amount: number;

	constructor(shoppingCart: ShoppingCart) {
		this.amount = DeliveryManager.deliveryCharge(shopingCart);
	}
}
```

이후 ShoppingCart클래스의 인스턴스 변수 products를 `private`로 변경해 외부에서의 위험을 없앤다.

```tsx
class ShoppingCart {
    private readonly products: Array<Product>;

    // 생략
}
```

DeliveryManager에서 사용하던 메서드 로직을 그대로 가져온다. (DeliveryManager를 지울거면 왜한거지??)

```tsx
class DeliveryCharge {
    readonly amount: number;

    constructor(shoppingCart: ShoppingCart) {
        if (shoppingCart.totalPrice() < 20000) {
            this.amount = 5000;
        } else this.amount = 0;
    }
}
```

이후에 20000, 5000, 0 과 같은 매직 넘버들은 이름을 붙여 상수로 관리한다.

```tsx
class DeliveryCharge {
    readonly amount: number;
    private static readonly CHARGE_FREE_THREASHOLD = 20000;
    private static readonly PAY_CHARGE = 5000;
    private static readonly CHARGE_FREE = 0;

    constructor(shoppingCart: ShoppingCart) {
        if (shoppingCart.totalPrice() < DeliveryCharge.CHARGE_FREE_THREASHOLD) {
            this.amount = DeliveryCharge.PAY_CHARGE;
        } else this.amount = DeliveryCharge.CHARGE_FREE;
    }
}
```

여기서 `amount`는 삼항 연산자로 바꾸는게 괜찮을 것 같다.

```tsx
class DeliveryCharge {
    readonly amount: number;
    private static readonly CHARGE_FREE_THREASHOLD = 20000;
    private static readonly PAY_CHARGE = 5000;
    private static readonly CHARGE_FREE = 0;

    constructor(shoppingCart: ShoppingCart) {
        this.amount =
            shoppingCart.totalPrice() < DeliveryCharge.CHARGE_FREE_THREASHOLD
                ? DeliveryCharge.PAY_CHARGE
                : DeliveryCharge.CHARGE_FREE;
    }
}
```

## 3. 불확실한 사양을 이해하기 위한 분석 방법

사양을 미리 알 수 있는 경우는 드물기에 ‘테스트가 없는 코드에 테스트를 추가해서 안전하게 리팩터링 할 수 있는 여러 가지 테크닉’이다.

### 사양 분석 방법 1: 문서화 테스트

```tsx
class MoneyManager {
	static calc(v: number; flag: boolean): number {
		// 생략
	}
}
```

메서드의 이름과 매개변수를 보고서는 어떤 의도인지 확실하게 유추할 수 없다.

```tsx
describe('문서화 테스트하기', () => {
    test('테스트 케이스 1', () => {
				const actual = MoneyManager.calc(1000. false);

        expect(actual).toBe(0);
				// 어떤 값이 나오는지 알았으니 맞게 변경한다.
				// expect(actual).toBe(1000);
    });
		// ...
});
```

몇가지 테스틀 더 진행해 다음과 같은 표를 얻을 수 있다.

| 매개변수 v | 매개변수 flag | 리턴 값 |
| ---------- | ------------- | ------- |
| 1000       | false         | 1000    |
| 2000       | false         | 2000    |
| 3000       | false         | 3000    |
| 1000       | true          | 1100    |
| 2000       | true          | 2200    |
| 3000       | true          | 3300    |

-   flag가 false라면, 그대로 리턴
-   flag가 true라면, 어떠한 계산을 한뒤 리턴

얼추 보면 10%를 계산해서 리턴하는 것으로 추측해볼 수 있다.

이러한 방법으로 완벽하게 밝히기는 어렵지만 이런식으로 분석할 수 있는 방법 중 하나로 보면 될 것 같다.

(여담이지만, 이런식으로 테스트 코드 작성해서 찾는게 맞을까? 싶다. 정말 규칙이 단순해서 그렇지 복잡하다면??? 그러면 이러한 불필요한 테스트코드를 통해 알아내는게 과연 맞을까 싶다)

### 사양 분석 방법 2: 스크래치 리팩터링

스크래치 리팩터링이란 로직의 의미와 구조를 분석하기 위해 시험 삼아 리팩터링 하는 것이다.

대상 코드를 체크아웃 한뒤에 테스트 코드를 작성하지 않고 리팩터링을 진행한다.

이런식으로 진행했을 때 몇가지 장점이 있다.

-   코드의 가독성이 좋아져 로직의 사양을 이해할 수 있게 된다.
-   이상적인 구조가 보인다.
-   쓸데없는 코드(데드 코드)가 보인다.
-   테스트 코드를 어떻게 작성해야 할지 보인다.

스크래치 리팩터링은 오로지 분석용이므로 끝난뒤에는 파기해야 한다.

## 4. IDE의 리팩터링 기능

### 리네임(이름 변경)

IDE에는 클래스, 메서드, 변수 등 이름을 전부 변경하는 기능이 들어 있다.

### 메서드 추출

로직의 일부를 메서드로 추출해주는 기능이다.

프론트에서도 긁어서 컴포넌트로 분리하던지 함수로 빼던지 다양한 기능을 에디터에서 제공해준다.

## 5. 리팩터링 시 주의 사항

### 기능 추가와 리팩터링 동시에 하지 않기

두 개의 모자(기능 추가 모자, 리팩터링 모자) 중에 하나만 쓰고 있어야 한다.

두가지를 같이 진행하게 되면 버그가 발생했을 때, 어떤 원인 때문에 버그가 발생했는지 찾기가 힘들다.

### 작은 단위로 시작하기

리팩터링은 작은 단계로 실시하는 것이 좋다.

예를 들어 메서드 이름 변경과 로직 이동을 했으면, 커밋을 서로 다르게 구분하는 것이 좋다.

### 불필요한 사양은 제거 고려하기

방해되는 코드가 불필요한 사양과 관련된 코드라면, 대처 방안을 검토하는 비용도 낭비라고 할 수 있다.

따라서 리팩터링 전에 불필요한 사양이 있는지, 미리 확인하는 것도 좋은 방법이다.
