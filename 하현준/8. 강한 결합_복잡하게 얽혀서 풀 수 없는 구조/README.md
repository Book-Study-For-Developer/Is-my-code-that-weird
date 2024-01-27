## 1. 결합도와 책무

아래 코드는 온라인 쇼핑몰에 할인 서비스가 추가된 코드이다.

- 상품 하나당 3000원 할인
- 최대 200000원까지 상품 추가 가능

```tsx
class DiscountManager {
  discountProducts: Array<Product> = [];
  totalPrice: number = 0;

  add(product: Product, productDiscount: ProductDiscount) {
    if (product.id < 0) {
      throw new Error();
    }
    if (product.name.length < 0) {
      throw new Error();
    }
    if (product.price < 0) {
      throw new Error();
    }
    if (product.id !== productDiscount.id) {
      throw new Error();
    }

    let discountPrice = DiscountManager.getDiscountPrice(product.price);

    let tmp;
    if (productDiscount.canDiscount) {
      tmp = this.totalPrice + discountPrice;
    } else {
      tmp = this.totalPrice + product.price;
    }

    if (tmp <= 200000) {
      this.totalPrice = tmp;
      this.discountProducts.push(product);
      return true;
    } else {
      return false;
    }
  }

  // 할인된 상품 가격 확인하기
  static getDiscountPrice(price: number) {
    let discountPrice = price - 3000;
    if (discountPrice < 0) discountPrice = 0;

    return discountPrice;
  }
}

class Product {
  id: number = 0;
  name: string = "";
  price: number = 0;
}

class ProductDiscount {
  id: number = 0;
  canDiscount: boolean = false;
}
```

이후 새로운 요구사항이 추가되어 다음과 같이 클래스를 구현했다.

- 300000까지 상품 추가 가능

```tsx
class Product {
  id: number = 0;
  name: string = "";
  price: number = 0;
  canDiscount: boolean = false; // 새로 추가함
}
class SummerDiscountManager {
  discountManager!: DiscountManager;

  add(product: Product) {
    if (product.id < 0) {
      throw new Error();
    }
    if (product.name.length === 0) {
      throw new Error();
    }

    let tmp;
    if (product.canDiscount) {
      tmp = this.discountManager.totalPrice + DiscountManager.getDiscountPrice(product.price);
    } else {
      tmp = this.discountManager.totalPrice + product.price;
    }

    if (tmp < 300000) {
      this.discountManager.totalPrice = tmp;
      this.discountManager.discountProducts.push(product);
      return true;
    } else {
      return false;
    }
  }
}
```

### 다양한 버그

할인 가격을 3000 → 4000으로 변경했다고 하자

그럼 다음과 같이 코드가 변경되는데 이렇게 되면 `SummerDiscountManager` 에도 영향을 미치게 된다.

```tsx
static getDiscountPrice(price: number) {
    let discountPrice = price - 4000;
    if (discountPrice < 0) discountPrice = 0;

    return discountPrice;
  }
```

### 로직의 위치에 일관성이 없음

- DiscountManager가 상품 정보 확인 말고도 할인 가격 계산, 할인 적용 여부 판단, 총액 상한 확인 등 많은 일을 담당
- 유효성 검사 로직이 각각 클래스에 구현되어 있음
- 일반 할인과 여름 할인의 구분이 어려움(ProductDiscount.canDsicount, Product.canDiscount)

위와 같은 클래스 설계가 바로 **책무를 고려하지 않은 설계**이다.

### 단일 책임 원칙

소프트웨어에서의 책임: 자신의 관심사와 관련해서, 정상적으로 동작하도록 제어하는 것

‘클래스가 담당하는 책임은 하나로 제한해야 한다’는 설계 원칙이다.

### 단일 책임 원칙 위반으로 발생하는 악마

`DiscountManager.getDiscountPrice`는 일반 할인 가격 계산을 책임지는 메서드이지 여름 할인 가격을 책임지기 위한 메서드가 아니다.

할인되는 가격이 같다고 해서 같은 메서드를 사용하면 스펙이 변경될 때 다른 쪽에도 같이 영향을 받게 된다.

### 책임이 하나가 되게 클래스 설계하기

상품의 가격을 나타내는 RegularPrice클래스를 만들고 유효성 검사 과정도 추가하여 가격과 관련된 책임을 넘겨서 중복되는 코드를 없앤다.

```tsx
class RegularPrice {
  private static readonly MIN_AMOUNT = 0;
  readonly amount: number;

  constructor(amount: number) {
    if (amount < RegularPrice.MIN_AMOUNT) {
      throw new Error("가격은 0 이상이어야 합니다.");
    }
    this.amount = amount;
  }
}
```

할인 클래스를 이제 변경한다.

```tsx
class RegularDiscountedPrice {
  private static readonly MIN_AMOUNT = 0;
  private static readonly DISCOUNT_AMOUNT = 4000;
  readonly amount: number;

  constructor(price: RegularPrice) {
    let discountedAmount = price.amount - RegularDiscountedPrice.DISCOUNT_AMOUNT;
    if (discountedAmount < RegularDiscountedPrice.MIN_AMOUNT) {
      discountedAmount = RegularDiscountedPrice.MIN_AMOUNT;
    }
    this.amount = discountedAmount;
  }
}

class SummerDiscountedPrice {
  private static readonly MIN_AMOUNT = 0;
  private static readonly DISCOUNT_AMOUNT = 3000;
  readonly amount: number;

  constructor(price: RegularPrice) {
    let discountedAmount = price.amount - SummerDiscountedPrice.DISCOUNT_AMOUNT;
    if (discountedAmount < SummerDiscountedPrice.MIN_AMOUNT) {
      discountedAmount = SummerDiscountedPrice.MIN_AMOUNT;
    }
    this.amount = discountedAmount;
  }
}
```

관심사에 따라 분리하여 독립되어 있는 구조로 구성했고 이를 **느슨한 결합**이라고 한다.

### DRY 원칙의 잘못된 적용

변경한 클래스에서는 DISCOUNT_AMOUNT를 제외하고 모든 로직이 같다. 그렇다고 해서 중복 코드를 작성했다고 생각하고 합치게되면 하나로 모인 로직이 여러 책무를 담당하게 된다.

DRY원칙은 ‘반복을 피하라’ 라는 의미로 ‘코드 중복을 절대 허용하지 말라’ 로 해석될 수 있지만 그것은 아니다.

‘모든 **지식**은 시스템 내에서 단 한번만, 애매하지 않고, 권위 있게 표현되어야 한다’

일반 할인과 여름 할인은 서로 다른 개념이며, DRY 원칙은 **각각의 개념 단위 내에서 반복을 하지 말라는 의미**이다.

즉 **같은 로직, 비슷한 로직이라도 개념이 다르면 중복을 허용해야 한다.**

## 2. 다양한 강한 결합 사례와 대처 방법

### 상속과 관련된 강한 결합

상속은 주의해서 다루지 않으면 강한 결합 구조를 유발하는 문법이다.

**슈퍼 클래스 의존**

다음과 같이 `PhysicalAttack` 클래스를 상속받아 구현된 클래스가 있다.

```tsx
class PhysicalAttack {
  singleAttackDamage(): number {
    return 1;
  }
  doubleAttackDamage(): number {
    return 2;
  }
}

class FighterPhysicalAttack extends PhysicalAttack {
  singleAttackDamage() {
    return super.singleAttackDamage() + 20;
  }
  doubleAttackDamage() {
    return super.doubleAttackDamage() + 10;
  }
}

const fighter = new FighterPhysicalAttack();
console.log(fighter.doubleAttackDamage()); // 12;
```

만약 PhysicalAttack의 doulbeAttackDamage가 singleAttackDamage를 두번 실행하는 메서드로 바뀔 경우 FighterPhysicalAttack 에서는 이제 다르게 계산이 된다.

```tsx
class PhysicalAttack {
  singleAttackDamage(): number {
    return 1;
  }
  doubleAttackDamage(): number {
    return this.singleAttackDamage() + this.singleAttackDamage();
  }
}

class FighterPhysicalAttack extends PhysicalAttack {
  singleAttackDamage() {
    return super.singleAttackDamage() + 20;
  }
  doubleAttackDamage() {
    return super.doubleAttackDamage() + 10;
  }
}

const fighter = new FighterPhysicalAttack();
console.log(fighter.doubleAttackDamage()); // 52;
```

super.doubleAttackDamage() 를 실행하면 singleAttackDamage()가 두번실행되는데 이때의 **`this`** 는 FighterPhysicalAttack 이므로 21을 반환한다 따라서 21 + 21 + 10 이 되어 52가 반환되게 되는 버그가 발생한다.

**상속보다 컴포지션**

상속을 사용하기 보단 컴포지션을 사용하는 것이 좋다.

```tsx
class PhysicalAttack {
  singleAttackDamage(): number {
    return 1;
  }
  doubleAttackDamage(): number {
    return this.singleAttackDamage() + this.singleAttackDamage();
  }
}

class FighterPhysicalAttack {
  private physicalAttack: PhysicalAttack;

  constructor(physicalAttack: PhysicalAttack) {
    this.physicalAttack = physicalAttack;
  }

  singleAttackDamage() {
    return this.physicalAttack.singleAttackDamage() + 20;
  }
  doubleAttackDamage() {
    // console.log(super.doubleAttackDamage())
    return this.physicalAttack.doubleAttackDamage() + 10;
  }
}

const test = new FighterPhysicalAttack(new PhysicalAttack());
console.log(test.doubleAttackDamage()); // 12
```

컴포지션 구조로 로직을 변경하면 영향을 적게 받게 된다.

**상속을 사용하는 나쁜 일반화**

상속을 사용하면 슈퍼 클래스가 공통 로직을 두는 장소로 사용된다. 그래서 무리하게 일반화하려고 하면 강한 결합이 발생되기 쉽다.

```tsx
abstract class DiscountBase {
  protected price: number;
  constructor(price: number) {
    this.price = price;
  }

  getDiscountedPrice() {
    let discountedPrice = this.price - 3000;
    if (discountedPrice < 0) {
      discountedPrice = 0;
    }
    return discountedPrice;
  }
}

class RegularDiscount extends DiscountBase {
  getDiscountedPrice() {
    let discountedPrice = this.price - 4000;
    if (discountedPrice < 0) {
      discountedPrice = 0;
    }
    return discountedPrice;
  }
}

class SummerDiscount extends DiscountBase {}

const test1 = new RegularDiscount(10000);
const test2 = new SummerDiscount(10000);

console.log(test1.getDiscountedPrice()); // 6000
console.log(test2.getDiscountedPrice()); // 7000
```

3000 에서 4000 만 다르므로 이렇게 작업할 수 있다. 여기서 더 나아가 로직이 같으므로 일반화를 위해 개선하려고 한다면 이렇게 된다.

```tsx
abstract class DiscountBase {
  protected price: number;
  constructor(price: number) {
    this.price = price;
  }

  getDiscountedPrice(): number {
    let discountedPrice = this.price - discountCharge();
    if (discountedPrice < 0) {
      discountedPrice = 0;
    }
    return discountedPrice;
  }
  protected discountedCharge() {
    return 3000;
  }
}

class RegularDiscount extends DiscountBase {
  discountedCharge() {
    return 4000;
  }
}
// 여름의 할인조건이 제품당 5% 할인
class SummerDiscount extends DiscountBase {
  getDiscountedPrice() {
    return Math.floor(this.price * (1.0 - 0.05));
  }
}
```

물론 정상적으로 동작은 하지만, 로직이 흩어져 있고 서브클래스에서 일부의 메서드만 빌려와 사용하기 때문에 추적이 어려워 좋지 않다.

```tsx
abstract class DiscountBase {
  protected price: number;
  constructor(price: number) {
    this.price = price;
  }

  getDiscountedPrice(): number {
    if (this instanceof RegularDiscount) {
      let discountedPrice = this.price - 4000;
      if (discountedPrice < 0) {
        discountedPrice = 0;
      }
      return discountedPrice;
    } else {
      return Math.floor(this.price * (1.0 - 0.05));
    }
  }
}
```

슈퍼 클래스에서 `instanceof` 로 조건 분기 처리 하여 로직을 실행하고 있다.
비지니스 개념이 서로 분산되어 있어 캡슐화도 되지 않고 상속을 사용함에 있어 장점을 누리지 못하는 클래스가 됐다.

### 인스턴스 변수별로 클래스 분할이 가능한 로직

```tsx
// Bad
class Util {
  private reservationId: number;
  private viewSettings: ViewSettings;
  private mailMagazine: MailMagazine;

  cancelReservation() {
    // reservationID를 사용한 예약 취소 처리
  }

  darkMode() {
    // 다크모드 표시 전환 처리
  }

  beginSendMail() {
    // 메일 전송 처리
  }
}

// Good
class Reservation {
  private reservationId: number;
  cancel() {}
}

class ViewCustomizing {
  private viewSettings: ViewSettings;
  darkMode() {
    // 다크모드 표시 전환 처리
  }
}

class MailMagazineService {
  private mailMagazine: MailMagazine;
  beginSendMail() {
    // 메일 전송 처리
  }
}
```

Util 클래스 하나에 책임이 다른 메소드들 끼리 뭉쳐 있었는데, 각 메서드 별로 3개의 클래스로 분리하여 강한 결합 문제를 없앤다.

### 특별한 이유 없이 public 사용하지 않기

특별한 이유 없이 public을 붙이면 강한 결합구조가 될 수 있다.
_(자바스크립트에는 class에 private을 붙일 수 없기에 자바스크립트로 변환이 어렵기에 자바 코드 그대로 썼다.)_

```java

// 히트포인트 회복 클래스
public class HitPointRecovery {
	public class HitPointRecovery(final Member chanter, final int targetMemberId, final PositiveFeelings positiveFeelings) {
		// 회복량의 복잡한 계산식
	})
}

// 호감도 제어하는 클래스
public class PositiveFeelings {
	// 호감도
	// 호감도 증가시키기
	// 호감도 감소시키기
}
```

PositiveFeelings는 내부에서 숨겨진 요소로 화면에서 표시하지도 않고 외부에서 제어하고싶지도 않은 클래스이다. 그러나 이유없이 public으로 만들면 클래스끼리 결합이 일어나 강한 결합구조가 된다.

### private 메서드가 너무 많다는 것은 책임이 너무 많다는 것

```java
class OrderService {
	private calcDiscountPrice(price: number) {
		// 할인 가격 계산 로직
	}
	private getProductBrowsingHistory(userId: number) {
		// 최근 봄 상품 리스트를 확인하는 로직
	}
}
```

가격 할인과 최근 본 상품 리스트 확인은 주문과는 다른 책임이기에 서로 다른 클래스로 분리하는 것이 좋다.

### 높은 응집도를 오해해서 생기는 강한 결합

관련이 깊은 데이터와 논리를 한곳에 모은 응집도가 높은 클래스를 잘못 이해해서 강한 결합이 발생하는 경우도 있다.

```tsx
// Bad
class SellingPrice {
  readonly amount: number;

  constructor(amount: number) {
    if (amount < 0) {
      throw new Error("가격은 0 이상이어야 합니다.");
    }
    this.amount = amount;
  }

  // 판매 수수료 계산하기
  calcSellingCommission() {}

  // 배송비 계산하기
  calcDeliveryCharge() {}

  // 추가할 쇼핑 포인트 계산하기
  calcShoppingPoint() {}
}

// Good
class SellingCommision {
  constructor(sellingPrice: SellingPrice) {
    // 판매 수수료 계산하기
  }
}

class DeliveryCharge {
  constructor(sellingPrice: SellingPrice) {
    // 배송비 계산하기
  }
}
class ShoppingPoint {
  constructor(sellingPrice: SellingPrice) {
    // 추가할 쇼핑 포인트 계산하기
  }
}
```

판매 수수료와 배송비는 판매 가격과 관련이 깊을 것이라고 생각하여 하나의 클래스에 모아두었다.
이는 판매 가격과 다른 개념이 섞여있으므로 **강한 결합에 해당**한다.

**응집도가 높다는 개념을 염두해서 로직을 한곳에 모으려고 했지만 결과적으로 강한 결합 구조를 만드는 상황이 발생했다.**

따라서 생성자에 계산에 사용될 판매 가격(`sellingPrice`)를 전달해서 각 클래스로 분리해서 사용해야 한다.

### 스마트 UI

화면 표시를 담당하는 클래스 중에서 화면 표시와 직접적인 관련이 없는 책무가 구현되어 있는 클래스를 **스마트 UI**라고 한다.

스마트 UI는 화면 표시에 관한 책무와 그렇지 않은 책무가 강하게 결합되어 있기 때문에 변경이 어렵다. 그러므로 서로 다른 클래스로 분할하는 것이 좋다.

### 거대 데이터 클래스

수많은 인스턴스 변수를 가진 클래스를 **거대 데이터 클래스**라고 한다.
거대 데이터 클래스는 다양한 데이터를 가져서 수많은 유스케이스에서 사용되고 이는 전역 변수와 동일한 유형의 악마들을 불러온다.

### 트랜잭션 스크립트 패턴

메서드 내부에 일련의 처리가 하나하나 길게 작성되어 있는 구조([add 메서드](https://www.notion.so/8-ef9c27ab2c1b43ceb6a0219f58dccfaa?pvs=21))를 **트랜잭션 스크립트 패턴** 이라고 한다. 이를 남용하면 메서드하나에 거대한 로직을 갖게 되므로 응집도는 낮아지고 결합도는 강해질 수 있다.

### 갓 클래스

하나의 클래스 내부에 수천에서 수만줄의 로직을 담고 수많은 책임을 담당하는 로직이 섞여있는 클래스를 **갓 클래스**라고 한다.

어떤 로직과 관련이 있는지, 책무를 파악하기 어렵기 때문에 원인 추적도 어렵고 좋지 않다.

### 강한 결합 클래스 대처 방법

강한 결합을 해결하기 위해서는 단일 책임 원칙에 따라 설계하는 것이다. 책임별로 클래스를 분할해서 작성해야 한다.
